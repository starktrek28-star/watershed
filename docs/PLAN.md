# Watershed — Plan

A virtual apartment you walk around in. Each room has corners, and the objects in those
corners hold your real stuff: whiteboards you actually write on, bookshelves whose books
are your files, a TV that plays your videos.

## The one rule

**Walking is the character. Doing is you.**
You walk the character to an object and click it (or press E). The view goes full screen
and you are working directly: writing on the whiteboard, reading the PDF, editing
your notes. Press Esc and you're back in the apartment, standing where you were.

## Constraints

- **Everything is built in code, online.** No Blender, no downloaded 3D models, no game
  engine. The apartment is generated from code (boxes, planes, simple shapes, text drawn on
  textures).
- **Runs on a normal laptop.** Simple geometry, no heavy lighting or shadows. Works in any
  browser, including a phone.
- **No AI for now.** That comes later.

## The apartment

```
┌──────────────────────┬──────────────────────┐
│ STUDY                │ TINKERING            │
│  Math corner         │  Workbench           │
│  Physics corner      │  Pegboard (projects) │
│  Coding desk         │  Parts drawers       │
│  Reading nook        │  3D printer          │
├───── door ───────────┴──────── door ────────┤
│  hallway        (you start here)            │
├───── door ──────────────┬─────── door ──────┤
│ LIVING / ENTERTAINMENT  │ KITCHEN           │
│  TV + couch             │  Fridge (groceries)│
│  Music corner           │  Recipe shelf     │
│  Games shelf            │  Meal plan board  │
└─────────────────────────┴───────────────────┘
```

### What the objects do (example: Math corner)

| Object | In the world | When you click it (full screen) |
|---|---|---|
| Whiteboard | Shows your latest board | Drawing canvas: pen, colours, eraser, pages. Auto-saves. |
| Bookshelf | Book spines labelled with your file names | Click a book to open the file: PDF, image, video, text or code |
| Desk | A "recent" card: what you last opened or solved | Your notes for that corner, plus the recent history |
| Drop zone | Glowing tray on the shelf | Drag files in, and they appear as new books |

Every corner works the same way, so tinkering, living room and kitchen reuse it:
workbench = project notes + photos, pegboard = project cards, TV = video player /
watchlist, fridge = grocery checklist, recipe shelf = recipe files.

## How it's built

| Part | Choice | Why |
|---|---|---|
| 3D world | Three.js, apartment generated in code | Runs in the browser, no tools to install |
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
| 1 | **Walkable apartment** | 4 rooms + hallway with furniture, a character you move with WASD / click, camera follows, walls block you, room lights turn on as you enter, labels on corners |
| 2 | **Full-screen interactions** | Click whiteboard → draw for real; click a book → file opens full screen; Esc returns to the world |
| 3 | **Your real data** | Files and whiteboards saved on the server; drag files onto shelves; "recent" shows what you last did; add/rename corners |
| 4 | **Fill every room** | Tinkering, living room and kitchen corners working (projects, videos, groceries, recipes) |
| 5 | **Online** | Password login, hosting guide, Google Drive folder setup |
| later | Extras | AI assistant, search, sounds, day/night, phone controls polish |
