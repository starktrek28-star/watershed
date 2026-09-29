# Watershed — Plan

A modern villa on its own grounds that you walk around in: lawns, an open field, a
wind-down garden, and one big single-storey bungalow with a rooftop terrace. Each room has
corners, and the objects in those corners hold your real stuff: whiteboards you actually
write on, bookshelves whose books are your files, a TV that plays your videos.

## The one rule

**Walking is the character. Doing is you.**
You walk the character to an object and click it (or press E). The view goes full screen
and you are working directly: writing on the whiteboard, reading the PDF, editing
your notes. Press Esc and you're back in the villa, standing where you were.

## Constraints

- **Everything is built in code, online.** No Blender, no downloaded 3D models, no game
  engine. The villa and grounds are generated from code (boxes, planes, simple shapes,
  text drawn on textures).
- **Runs on a normal laptop.** Simple geometry, soft fake lighting, no heavy shadows.
  Trees and grass are repeated cheap shapes. Works in any browser, including a phone.
- **No AI for now.** That comes later.

## Architecture of the villa

A **modern tropical bungalow**: long flat roofs with deep overhangs, floor-to-ceiling glass
facing the lawns, stone and timber walls, and a **central courtyard** with a tree that
every wing looks onto. Built around the courtyard in a U-shape, so you are never more than
a few steps from the garden.

- **Ground floor:** everything you use daily.
- **Rooftop terrace** (the second level): open-air lounge and stargazing deck, reached by
  an outdoor stair from the courtyard.
- When you walk inside, the roof above you fades away (cutaway view), so you can always see
  your character and the room.

### Site plan

```
                         OPEN FIELD
          (long grass, wildflowers, a path to a lone tree + bench)
 ─────────────────────────────────────────────────────────────────
                        BACK LAWN
     ┌──────────────┐                        ┌──────────────────┐
     │ WIND-DOWN    │    lap pool + deck     │ FIREPIT CIRCLE    │
     │ GARDEN       │                        │ (evening seating) │
     │ hammock,     │                        └──────────────────┘
     │ reading chair│
     └──────────────┘
   ┌─────────────────────────────────────────────────────────────┐
   │                     THE BUNGALOW (below)                    │
   └─────────────────────────────────────────────────────────────┘
                        FRONT LAWN
              driveway ── entry path ── gate            (you arrive here)
```

### Bungalow floor plan

```
┌───────────────────┬───────────────────────────────┬─────────────────────┐
│ STUDY WING        │ GREAT ROOM (double height,     │ KITCHEN + DINING    │
│                   │ glass wall to back lawn)       │                     │
│  Math corner      │  Living / entertainment        │  Fridge (groceries) │
│  Physics corner   │  TV wall + sofa               │  Recipe shelf       │
│  Coding desk      │  Music corner                 │  Meal plan board    │
│  Library / reading│  Games shelf                  │  Island + dining    │
│  nook             │                               │  table              │
├──────── glass ────┤        ┌───────────────┐       ├──── glass ──────────┤
│ corridor          │        │   COURTYARD   │       │ corridor            │
│                   │        │  tree, water, │       │                     │
├───────────────────┤        │  stair to roof│       ├─────────────────────┤
│ TINKERING WORKSHOP│        └───────────────┘       │ BEDROOM SUITE       │
│  Workbench        │                               │ (quiet room,         │
│  Pegboard         │          ENTRY FOYER          │  later: journal,     │
│  Parts drawers    │        (front door, lawn)     │  sleep)              │
│  3D printer       │                               │                     │
│  roll-up door to  │                               │                     │
│  the yard         │                               │                     │
└───────────────────┴───────────────────────────────┴─────────────────────┘
```

### Outdoor wind-down places

| Place | What it's for |
|---|---|
| Wind-down garden | Hammock and reading chair. Click to open your "read later" shelf or a calm-music player |
| Firepit circle | Evening spot. Click for a daily journal / reflection page |
| Lap pool + deck | Pure scenery: just walk out and look at the field |
| Lone tree in the field | A bench at the end of a path, the quietest spot, for thinking |
| Rooftop terrace | Lounge + stargazing, the night-time hangout |

## What the objects do (example: Math corner)

| Object | In the world | When you click it (full screen) |
|---|---|---|
| Whiteboard | Shows your latest board | Drawing canvas: pen, colours, eraser, pages. Auto-saves. |
| Bookshelf | Book spines labelled with your file names | Click a book to open the file: PDF, image, video, text or code |
| Desk | A "recent" card: what you last opened or solved | Your notes for that corner, plus the recent history |
| Drop zone | Glowing tray on the shelf | Drag files in, and they appear as new books |

Every corner works the same way, so the other rooms reuse it:
workbench = project notes + photos, pegboard = project cards, TV = video player /
watchlist, fridge = grocery checklist, recipe shelf = recipe files.

## How it's built

| Part | Choice | Why |
|---|---|---|
| 3D world | Three.js, villa and grounds generated in code | Runs in the browser, no tools to install |
| Full-screen views | Plain HTML (PDF viewer, canvas, editor) | 2D stuff is easy on the web |
| Server | Small Node.js app | Saves your data and files |
| Storage | A `data/` folder: a database file + your files | Simple, easy to back up |
| Google Drive (5 TB) | Point `data/` at a Google Drive for Desktop folder | Syncs to Drive with no extra code |

## Hosting

1. **On your laptop:** `npm install`, `npm start`, open `http://localhost:3000`.
2. **From your phone / anywhere:** Tailscale (free, private) pointing at the laptop.
3. **Always online (later):** deploy the same app to a small host (Render / Railway / VPS)
   with a password.

## Phases

| # | Phase | Done when |
|---|---|---|
| 1 | **Villa + grounds** | Walk from the gate across the front lawn, into the foyer, through every wing, out to the back lawn, field and rooftop. Furniture in place, walls and stairs work, roof cutaway, room lights on as you enter, labels on corners, runs smoothly on your laptop |
| 2 | **Full-screen interactions** | Click whiteboard → draw for real; click a book → file opens full screen; Esc returns to the world |
| 3 | **Your real data** | Files and whiteboards saved on the server; drag files onto shelves; "recent" shows what you last did; add/rename corners |
| 4 | **Every room + outdoors working** | Tinkering, great room, kitchen, and wind-down spots (journal, read-later, music) working |
| 5 | **Online** | Password login, hosting guide, Google Drive folder setup |
| later | Extras | AI assistant, day/night sky, sounds, search, bedroom features |
