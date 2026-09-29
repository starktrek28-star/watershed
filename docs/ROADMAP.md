# Watershed — Delivery Roadmap

Eight phases. Each phase ships something production-grade and usable on its own, and ends
with an explicit **exit gate**: nothing moves to the next phase until the gate passes.
Testing, security, and ops are part of every phase, not a phase at the end.

Estimates assume one lead engineer working with AI assistance, part-time. They are sizing
guidance, not promises.

| Phase | Codename | Outcome | Est. |
|---|---|---|---|
| 0 | **Blueprint** | Design, art direction, repo, CI/CD, infra, auth | 2–3 wks |
| 1 | **Mark I** | The residence exists: walk it, feel it | 4–6 wks |
| 2 | **Arc Reactor** | Data core: API, DB, Drive storage, activity, search | 3–4 wks |
| 3 | **Workshop** | Rooms come alive: modules, panels, in-world displays | 5–7 wks |
| 4 | **J.A.R.V.I.S.** | Assistant: command, voice, briefing, semantic memory | 4–5 wks |
| 5 | **Uplink** | Integrations: Drive import, Calendar, GitHub, Spotify, printer, lights | 4–6 wks |
| 6 | **Hardening** | Security audit, perf, accessibility, DR, v1.0 release | 2–3 wks |
| 7 | **Mark II+** | VR, mobile companion, guests, new rooms | ongoing |

---

## Phase 0 — Blueprint

**Goal:** every later phase builds on solid ground.

Deliverables
- Product spec: rooms, stations, sections, and the core user journeys ("walk into the
  study and resume math", "log a finished print", "what should I cook tonight").
- **Floor plan** (to scale) and **art direction bible**: mood boards, palette, materials,
  lighting at day/night, HUD visual language (holo frames, typography, motion rules).
- Monorepo scaffold: `apps/residence`, `apps/api`, `apps/worker`, `packages/ui`,
  `packages/modules`, `packages/schema`, `infra/`.
- Tooling: TypeScript strict, ESLint, Prettier, Vitest, Playwright, commit conventions.
- CI: lint, typecheck, test, build, container images, preview deploys.
- Infra: Docker Compose (Postgres, API, worker, Caddy, observability), secrets management,
  deploy to the home server.
- Auth: passkey registration/login, sessions, TOTP fallback.
- Decide and record hosting hardware and access (Tailscale vs. Cloudflare Tunnel).

**Exit gate:** a "hello" page behind passkey login, deployed by CI to the home server,
reachable from the owner's phone; architecture and art bible signed off.

---

## Phase 1 — Mark I (the world)

**Goal:** the residence exists and feels premium, even before it holds data.

Deliverables
- Apartment modeled in Blender: study, workshop, living/media, kitchen, hallway/entry.
  Baked lightmaps, PBR materials, KTX2 textures, LODs.
- Character: rigged avatar with idle/walk/run/turn animations, Rapier character
  controller (walls, furniture, doors).
- Camera: third-person follow with collision (never clips through walls), first-person
  toggle, smooth transitions; walls fade when they block the view.
- Input: keyboard + mouse, click/tap-to-go, touch joystick, gamepad.
- **Presence system:** room detection drives lights turning on as you enter, ambient audio
  zones, room name reveal.
- Interaction system: stations with proximity prompts, focus highlight, "walk to" on click.
- Day/night cycle tied to real local time; windows show matching sky.
- Loading experience (boot sequence), adaptive quality, frame-budget monitor.
- Visual regression tests (Playwright screenshots of fixed camera shots).

**Exit gate:** 60 fps on the target laptop, 30 fps on the target phone, all stations
reachable, load < 4 s, owner walkthrough approved.

---

## Phase 2 — Arc Reactor (data core)

**Goal:** a reliable, typed home for all data, independent of the 3D client.

Deliverables
- Postgres schema (rooms, stations, sections, items, attachments, links, tags, activity,
  sessions) with Drizzle migrations.
- Core API (Fastify + Zod), OpenAPI spec, generated typed client.
- Storage abstraction with **Google Drive driver** (OAuth, resumable uploads, dedicated
  `Watershed/` folder tree, quota monitoring) plus local-disk driver for dev.
- File pipeline in the worker: thumbnails, PDF/image previews, text extraction/OCR for
  search.
- Activity log and "resume" queries (last session and recent activity per section).
- Full-text search across items and file contents.
- WebSocket channel for live updates across open clients.
- Backups: nightly encrypted DB dumps to Drive; automated restore test.

**Exit gate:** API at 100% schema coverage with integration tests; a 1 GB upload
round-trips through Drive; restore drill passes; p95 API latency < 150 ms on the home
server.

---

## Phase 3 — Workshop (rooms come alive)

**Goal:** every room shows and edits the owner's real data.

Deliverables
- HUD design system in `packages/ui`: holo panels, lists, editors, file viewer (PDF,
  images, video, code), dropzones, toasts, command palette shell.
- Module framework: registry, schema, panel, in-world widget, default stations.
- **Study:** Math (problems in LaTeX, attempts, time, solution scans), Physics, Coding,
  Reading (books, progress, highlights). Focus timer and streaks. Whiteboard renders the
  current problem in the world.
- **Workshop:** Projects (status, BOM, photos, notes), Electronics, 3D Printing (print
  log, files, settings), Parts inventory. Pegboard shows active projects.
- **Living / Media:** Watchlist (movies/shows), Music, Games. TV shows "up next".
- **Kitchen:** Recipes, Groceries, Meal plan. Fridge door shows the grocery list.
- Owner can create custom sections and place them on stations.
- Quick capture from anywhere (note, photo, file) with routing to a section.

**Exit gate:** the owner runs their real week in it for 7 days without falling back to
another tool for these areas; e2e tests cover every module's create/edit/attach/resume
flow.

---

## Phase 4 — J.A.R.V.I.S.

**Goal:** the residence talks back and helps.

Deliverables
- Command palette (Ctrl/⌘-K): navigate, search, create, jump to room.
- AI assistant on the Claude API with tool use scoped per module ("add eggs to groceries",
  "what was I stuck on in math?", "summarise this week's workshop progress").
- Semantic memory: embeddings in pgvector across items and extracted file text.
- Voice: wake-word or push-to-talk, speech-to-text, spoken replies; a distinct JARVIS
  voice and HUD presence.
- Daily briefing when the owner walks in: what's in progress, streaks, due items,
  calendar, printer status.
- Guardrails: confirmation for destructive actions, cost and rate limits, audit log of
  every AI action.

**Exit gate:** an eval suite of 50+ real requests passes at ≥ 95% correct tool use; no
destructive action without confirmation; voice round trip < 2 s.

---

## Phase 5 — Uplink (integrations)

**Goal:** the residence reflects the owner's outside world automatically.

Deliverables
- **Google Drive import:** map existing Drive folders to sections, incremental sync.
- Google Calendar: schedule in the briefing, study blocks.
- GitHub: Coding section shows repos, recent commits, open PRs.
- Spotify: now playing in the living room; control from the room.
- TMDB: posters and metadata for the watchlist.
- 3D printer (OctoPrint / Moonraker): live print progress on the printer model in the
  workshop, camera snapshot, finish notification.
- Home Assistant (optional): real lights follow the virtual room you're in, and back.
- Integration framework: OAuth vault, webhooks, polling jobs, per-integration health.

**Exit gate:** each integration has health monitoring, token refresh, failure alerts,
and a documented disconnect/purge path.

---

## Phase 6 — Hardening → v1.0

**Goal:** a release the owner can rely on for years.

Deliverables
- External-style security review: auth, uploads, SSRF in integrations, AI prompt
  injection via stored content, dependency audit.
- Performance pass: asset budgets, memory leaks over long sessions, battery on mobile.
- Accessibility: keyboard-only navigation, screen-reader "list mode" of every room,
  reduced motion, captions for JARVIS voice.
- Disaster recovery runbook: rebuild the whole server from Drive backups in < 1 hour.
- Offline/PWA: companion works read-only offline and syncs queued writes.
- Documentation: owner's guide, ops runbook, module authoring guide.

**Exit gate:** security findings closed, DR drill < 1 h, 30-day burn-in with no
data-loss incidents → **tag v1.0**.

---

## Phase 7 — Mark II and beyond

- WebXR: walk the residence in a VR headset.
- Mobile companion app (native shell around the PWA) with widgets and share-sheet capture.
- Guest mode: invite someone into a room with scoped read access.
- New rooms: gym (workouts), garage, bedroom (sleep, journal), vault (documents).
- Analytics lab: long-term trends (study hours, projects shipped, meals cooked).

---

## Working agreement

- Every phase runs as small PRs to `main` with CI green and preview deploys.
- Each phase starts with a short design review and ends with a demo against its exit gate.
- Decisions that are expensive to reverse get an ADR in `docs/ARCHITECTURE.md`.

## Decisions needed from the owner before Phase 0 closes

1. Hosting hardware: an existing laptop, a dedicated mini PC (recommended), or a VPS.
2. Access: private only (Tailscale) or a public URL with login (Cloudflare Tunnel).
3. AI budget for J.A.R.V.I.S. (monthly Claude API cap).
4. Hardware to integrate: 3D printer model and firmware, smart lights, and so on.
5. Target devices: laptop model and phone model (these set the performance budgets).
