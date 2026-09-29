# Watershed — System Architecture

> A walkable residence where every room is a live interface to the owner's real life:
> study, workshop, media, kitchen. Walk up to the whiteboard, see the problem you were
> solving last night. Walk up to the printer, see the print that's 64% done.

This document records the architecture and the decisions behind it. Decisions that are
expensive to reverse are captured as ADRs (see the end of this file).

---

## 1. Design principles

1. **The world is the interface, not decoration.** Data shows up *on* objects
   (diegetic displays), not only in pop-up panels. The whiteboard shows the current
   equation; the fridge door shows the grocery count; the TV shows what's next.
2. **2D where 2D is better.** Reading, writing, and file browsing happen in crisp HTML
   panels layered over the 3D world. We never fight a game engine to render a text box.
3. **Data outlives the front end.** All data lives behind a typed API. The 3D client is
   one consumer; a phone companion, a CLI, voice, or a future Unity/VR client are others.
4. **Owner's data stays owner's.** Self-hosted by default. Files live in the owner's
   Google Drive. No third party holds the database.
5. **Production from day one.** Typed end to end, tested, observable, backed up,
   reproducible deploys. No "we'll fix it later" in the foundations.
6. **Extensible by module.** A room section (Math, Recipes, 3D Printing) is a module that
   declares its schema, panel UI, and in-world widget. New sections don't touch the core.

---

## 2. System overview

```
┌──────────────────────────── Clients ─────────────────────────────┐
│  Residence (web, 3D)      Companion (mobile PWA)     Voice / CLI │
│  React + R3F + Rapier     same UI kit, no 3D         JARVIS only │
└───────────────┬───────────────────────┬──────────────────┬───────┘
                │ HTTPS (REST + OpenAPI) │ WebSocket (live) │
┌───────────────▼───────────────────────▼──────────────────▼───────┐
│                        Core API (Fastify, TS)                    │
│  auth · rooms/sections · items · files · activity · search       │
│  module registry · integrations · JARVIS orchestration           │
├──────────────┬──────────────┬──────────────┬─────────────────────┤
│ PostgreSQL   │ Job runner   │ Storage      │ AI gateway          │
│ + pgvector   │ (pg-boss)    │ abstraction  │ Claude API (tools), │
│ FTS, events  │ thumbnails,  │ ├ Google     │ speech-to-text,     │
│              │ OCR, embeds, │ │  Drive     │ text-to-speech      │
│              │ sync, backup │ ├ local disk │                     │
│              │              │ └ S3 (opt.)  │                     │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
        ▲ integrations: Drive, Calendar, GitHub, Spotify, TMDB,
        │ OctoPrint/Moonraker (3D printer), Home Assistant (real lights)
```

### Deployment topology

```
Home server (mini PC / NUC, Docker Compose)          Anywhere
┌─────────────────────────────────────────┐   ┌──────────────────┐
│ caddy (TLS) → web (static) + api        │◄──┤ Tailscale (private)│
│ postgres · worker · glitchtip · grafana │   │ or Cloudflare Tunnel│
└───────────────┬─────────────────────────┘   └──────────────────┘
                │ nightly encrypted backups
                ▼
         Google Drive (5 TB): files + DB backups
```

The same Compose file runs on the owner's laptop for development and on a VPS if the
owner ever prefers cloud hosting.

---

## 3. Technology stack

| Layer | Choice | Why |
|---|---|---|
| Language | TypeScript everywhere | One language, shared types between client, server, modules |
| Monorepo | pnpm workspaces + Turborepo | Shared packages, cached builds |
| 3D | Three.js via React Three Fiber + drei | Mature web 3D; React makes 2D HUD + 3D one codebase |
| Physics / movement | Rapier (`@react-three/rapier`) | Real collisions, character controller, stairs/slopes |
| HUD / panels | React, Radix primitives, Tailwind, Framer Motion | Accessible, animated "holographic" UI |
| Client state | Zustand (world), TanStack Query (server data) | Clear split: sim state vs. server cache |
| Art pipeline | Blender → glTF, baked lightmaps, KTX2 textures, meshopt | Film-quality lighting at game-level cost |
| API | Fastify + Zod + OpenAPI generation | Fast, schema-first, typed client generated |
| Database | PostgreSQL 16 + Drizzle ORM + pgvector | Relational core, full-text + semantic search in one DB |
| Jobs | pg-boss | Durable queues without adding Redis |
| Realtime | WebSocket (Fastify) | Live updates across devices, printer status, etc. |
| Files | Storage driver interface: Google Drive (primary), local, S3 | Use the 5 TB; swappable |
| Auth | Passkeys (WebAuthn) + TOTP fallback, HTTP-only sessions | Owner-grade security, no passwords to leak |
| AI (JARVIS) | Claude API with tool use; Whisper-class STT; neural TTS | Assistant that can read and act on the residence's data |
| Observability | pino logs, OpenTelemetry, Grafana/Prometheus, GlitchTip | Know when something breaks before the owner does |
| Tests | Vitest, Playwright (e2e + visual), k6 (load) | Unit → integration → scene screenshots |
| CI/CD | GitHub Actions → container images → deploy to home server | Every merge is deployable |

---

## 4. Domain model

```
Residence
 └─ Room            (study, workshop, living, kitchen, …)
     └─ Station     (a 3D anchor: whiteboard, workbench, fridge)
         └─ Section (Math, Electronics, Recipes …) — typed by a Module
             └─ Item        (typed record: problem, book, project, recipe, title …)
                 ├─ Attachment → File (stored via storage driver)
                 ├─ Link       (URL, Drive file, GitHub repo …)
                 └─ Tag
Activity  (append-only event log: created, updated, solved, uploaded, watched …)
Session   (focused work session: section, start, end, notes — powers "recently")
```

- **Module**: declares `kind`, a Zod schema for its items, list/detail panel components,
  an optional in-world widget, JARVIS tools, and default stations. Examples:
  - `math.problems` — statement (LaTeX), source, difficulty, status, attempts, time spent,
    solution photo/PDF.
  - `reading.books` — title, author, progress %, highlights.
  - `workshop.projects` — BOM, status, photos, linked parts, linked prints.
  - `media.watchlist` — TMDB-backed titles, status, rating.
  - `kitchen.recipes` / `kitchen.groceries` / `kitchen.mealplan`.
- **"What was I doing?"** is a first-class query: last active Session and most recent
  Activity per section, shown the moment the owner walks up to a station.

---

## 5. Client architecture

```
apps/residence
 ├─ world/        scene graph, rooms (glTF), lighting, audio zones
 ├─ player/       character controller, camera rig (3rd / 1st person), input (kb, mouse, touch, gamepad)
 ├─ interaction/  stations, proximity + focus, prompts, click-to-go
 ├─ presence/     room detection → lights, ambient audio, JARVIS context
 ├─ hud/          design system: panels, holo frames, command palette, notifications
 ├─ modules/      per-section panels + in-world widgets (loaded from packages/modules)
 └─ perf/         LOD, adaptive quality, frame budget monitor
```

- The 3D canvas and the HUD share one React tree; panels are DOM, widgets inside the world
  render to textures (e.g. KaTeX → canvas → whiteboard material).
- Movement and camera stay responsive while panels are open (panels pause input only when
  focused).
- **Performance budgets:** 60 fps on an integrated-GPU laptop at 1080p; 30 fps on a
  mid-range phone; first interactive < 4 s on broadband; < 40 MB compressed assets.

---

## 6. Security

- Single-owner system with optional scoped guest access later.
- Passkeys for login; sessions in HTTP-only, SameSite=strict cookies; CSRF protection.
- Private by default through Tailscale; public exposure only through Cloudflare Tunnel +
  Access policy.
- Google OAuth tokens encrypted at rest (libsodium, key outside the DB).
- File uploads: size limits, MIME sniffing, served from an isolated path with
  `Content-Disposition` and strict CSP.
- JARVIS tools are allow-listed per module; destructive actions require confirmation.
- Dependency scanning and secret scanning in CI.

---

## 7. Reliability & operations

- Nightly `pg_dump` + file manifest, encrypted, pushed to Google Drive; weekly restore
  drill in CI against a scratch database.
- Migrations are forward-only and run automatically on deploy with a pre-deploy backup.
- Health checks, uptime alerts, error tracking, dashboards for API latency, job queue
  depth, Drive quota, disk, and client frame rate.
- Offline tolerance: the companion PWA caches recent items and queues writes.

---

## 8. Architecture Decision Records

| # | Decision | Status |
|---|---|---|
| ADR-001 | Web (Three.js/R3F) over Unity/Godot: 2D-heavy UI, any-device access, single codebase | Accepted |
| ADR-002 | Self-hosted Postgres as system of record; Google Drive as blob store only | Accepted |
| ADR-003 | Modules own their item schema, UI, widget, and AI tools | Accepted |
| ADR-004 | Passkeys first, no passwords | Accepted |
| ADR-005 | Stylized-realistic art direction with baked lighting (not photoreal, not low-poly) | Accepted |
| ADR-006 | Third-person default camera, first-person toggle | Accepted |
| ADR-007 | Private access via Tailscale by default | Proposed |
