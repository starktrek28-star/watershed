# Handoff: PMKSY Multi-User District Reporting Portal

## Overview
A multi-user data collection and reporting system for Karnataka's PMKSY-WDC 2.0 / PMKSY-OI watershed development programme. Replaces the manual R-1 to R-12 spreadsheet returns collected from ~30 districts each month. Two roles: **District Officer** (fills only their district's data) and **State Admin** (sees everything, manages reporting periods, consolidates and exports).

This bundle is the product/UX spec. Claude Code's job is to **rebuild this as a real web app with a backend** — a database, authentication, and permissions — not to ship the bundled HTML as-is.

## About the Design Files
The file `PMKSY Reporting Portal.dc.html` in this folder is a **design reference/prototype** built in HTML with an in-memory data model (resets on reload, no real accounts). It demonstrates the intended screens, layout, data shapes, and interaction flow. Recreate it in whatever stack you choose (recommended: a standard web stack — e.g. Next.js/React frontend + a real database — but pick what fits your environment) with a real backend behind it. Treat this as **high-fidelity** for layout/spacing/copy/status colors, but the "backend" in the prototype (state seeding, simulated exports) is illustrative only and must be replaced with real logic.

## Fidelity
**High-fidelity** for screen layout, typography scale, spacing, color system, and copy. **Not** high-fidelity for data persistence, auth, or export generation — those are simulated in the prototype and need real implementations.

## Required Backend (this is the core of the work)
- **Auth**: two roles — District Officer (scoped to exactly one district) and State Admin (sees all districts). Real login (username/password or SSO), session/JWT-based.
- **Database** tables (see Data Model below): districts, users, reporting_periods, report_formats (the 11 tables), submissions, entries.
- **Authorization**: District users can only read/write their own district's submissions, and only when the active period is `open` AND their submission status isn't `submitted`. Admins can read/write everything, lock/reopen periods, and (recommended addition) reopen an individual district's submission back to draft.
- **Real Excel/PDF export**: server-side generation (e.g. a library like ExcelJS / openpyxl-equivalent, and a PDF renderer) producing the same district-wise + Total-row + Grand-Total layout shown in the consolidated report screen.

## Screens / Views
### 1. Login
- Split-card layout: left panel (dark navy `oklch(24% 0.055 250)`, white text) with programme branding; right panel with role toggle (District Officer / State Admin), district dropdown (district role only), username, password, Sign In button.
- Real version: replace with real credential check against the users table; district officers should only ever see/select their own assigned district (not a free dropdown), typically pre-bound to their account.

### 2. District Dashboard
- Header: portal name, district name badge, role label, logout.
- Body: "Reporting Period: March 2026" label (single active period), Export All (Excel/PDF) buttons, and a responsive grid of cards — one per report format — each showing format title, status chip (Not Started / Draft / Submitted), entry count, and a "Fill Report / Continue Filling / View" button.
- If the active period is locked by admin, show a banner and make everything read-only.

### 3. Report Form (one format × one district × the active period)
- Header: format code/short label, period, full title, status chip, Export Excel/PDF buttons.
- Table of existing entries (columns depend on the selected format — see Data Model); Edit/Remove actions per row.
- "+ Add Entry" reveals an inline form with one input per column (text or number), Add/Update + Cancel buttons.
- Footer: Save Draft, Submit Report. Once submitted (or period locked), the whole screen becomes read-only with an explanatory note.
- **Special case — Format R1 "Physical Achievements"**: this format's table has a genuine two-level header from the source PDF — group headers spanning multiple sub-columns (see "R1 Column Groups" below). Recreate this exact grouped header (`colSpan`/`rowSpan`), not a flat single-row header. All other 10 formats use a simple flat header (their columns are currently simplified placeholders — see Known Gaps).

### 4. State Admin Dashboard
- Header: portal name, "State Admin" badge, logout.
- Reporting Periods panel: list of periods as pills (label + Open/Locked state + Lock/Reopen toggle) and a "+ Create Period" control (text input + Create button).
- **Two pie charts** ("Entries by Table", "Entries by District"): CSS `conic-gradient` donut + scrollable color-swatch legend (label + entry count + percentage), each sorted largest-value-first. Metric = total entries recorded, across all districts for a table / across all tables for a district, for the active period.
- Submission Status matrix: districts (rows) × formats (columns), each cell a small colored square (green = submitted, amber = draft, gray = not started) with a tooltip; clicking a column header or cell jumps to that format's consolidated report.
- Consolidated Reports by Format: one row per format with title, "`X / 30` districts submitted", View link, Export Excel/PDF buttons.

### 5. Admin Consolidated Report (per format)
- Header: format label/period, title, Export Excel/PDF buttons.
- Table: for each district with entries, its rows (district name shown once, on the first row) followed by a bold "Total" subtotal row (sum of that district's numeric columns), then a final bold "Grand Total" row across all districts. This mirrors the original PDF's per-district Total + overall Grand Total structure.
- Format R1 uses the same two-level grouped header as its data-entry screen.

## Data Model
```
District:        id, name                                   (30 real Karnataka districts, see DISTRICTS in the prototype's script)
User:             id, username, passwordHash, role ('district'|'admin'), districtId (null for admin)
ReportingPeriod:  id, label (e.g. "March 2026"), status ('open'|'locked')
ReportFormat:     id ('r1'..'r11'), title (full real name), shortLabel, columns: [{ key, label, type: 'text'|'number' }]
Submission:       id, periodId, districtId, formatId, status ('not_started'|'draft'|'submitted'), submittedAt
Entry:            id, submissionId, <one column per ReportFormat.columns> (JSON blob or a dynamic/EAV table since column sets differ per format)
```
Uniqueness: one `Submission` per (periodId, districtId, formatId). `Entry` rows belong to a submission; "Save Draft" keeps status at `draft` (or promotes `not_started`→`draft`), "Submit Report" sets `submitted` (locks district editing until admin reopens it — recommend adding an explicit per-submission "Reopen" action for admins, not just period-level lock/unlock).

### The 11 Report Formats (exact titles, from the source PDF)
1. **R1** — District-wise Physical Achievements — Jal Shakthi Abhiyan (PMKSY-WDC 2.0) — *fully modeled, see R1 Column Groups below*
2. **OOMF** — Output Outcome Monitoring Framework (OOMF) under PMKSY-WDC 2.0 — columns: indicator (text), target (number), achievement (number)
3. **Component Progress** — PMKSY-WDC 2.0 Componentwise Physical and Financial Progress — columns: component (text), physicalPct (number), financialLakhs (number)
4. **Financial Progress** — PMKSY-WDC 2.0 Financial Progress (From Inception to Till Reporting Month) — columns: project (text), sanctioned, released, expenditure, balance (all number, Rs Lakhs)
5. **VC Live Financial** — VC Live 2025-26 Project and Component Wise Financial Progress under PMKSY-WDC 2.0 (Rs in Lakhs) — columns: project (text), component (text), released, expenditure (number)
6. **SCP/TSP** — PMKSY-WDC 2.0 Componentwise-SCP/TSP — columns: component (text), scp, tsp (number, Rs Lakhs)
7. **OI Physical/Financial** — VC Live PMKSY-OI: Componentwise Physical & Financial Progress as on Date — columns: component (text), physicalPct, financialLakhs (number)
8. **OI Treasury** — PMKSY-OI Treasury Progress and Balance Funds with Districts — columns: scheme (text), released, utilized (number, Rs Lakhs)
9. **OI SCSP/TSP** — PMKSY-Other Interventions (PMKSY-OI) — SCSP/TSP — columns: component (text), scsp, tsp (number, Rs Lakhs)
10. **MGNREGA Convergence** — Convergence of PMKSY-WDC 2.0 Programme with MGNREGA, PMKSY-OI, RKVY, ABY & Other Programmes — columns: scheme (text), activities (number), amount (number, Rs Lakhs)
11. **Protected Area Convergence** — Convergence of PMKSY-WDC 2.0 Programme with MGNREGA in WDC 2.0 Protected Areas Only — columns: activity (text), works, persondays (number)

**Known gap to flag to the user/PM**: formats 2–11 currently have simplified/representative columns, not verified against the full PDF (only R1 was fully extracted and verified against source data). Before building the backend schema for those 10 formats, either re-extract their real column structures from the source PDF or confirm the simplified structure is acceptable.

### R1 Column Groups (exact PDF two-level header — implement this precisely)
Leading columns (no group, span both header rows): **Project Name** (text), **Block** (text).
Then, grouped:
| Group header | Sub-columns (all numeric) |
|---|---|
| Water Conservation & Rainwater Harvesting — Newly Constructed | Amrit Sarovar/Nala Bund/Gokatte (Nos), Check Dam/Vented Dam (Nos), Farm Pond (Nos), Percolation Tank/Mini Percolation Tank etc (Nos), Rainwater Recharge Structures (Gully Plugs, Open Well Recharge, Sand Filter) (Nos), Other Water Conservation Structures (Nos) |
| Renovation of Traditional (Old) Water Bodies / Tanks Rejuvenated | Amrit Sarovar/Nala Bund/Gokatte (Nos), Check Dam/Vented Dam etc (Nos), Farm Pond (Nos), Percolation Tank/Mini Percolation Tank etc (Nos), Ground Water Recharge Structure (Nos), Others - Traditional Water Bodies Restored (Nos) |
| Soil & Moisture Conservation Measures (Watershed Development) | Staggered Trenches (ha), Bench Terracing (ha), Contour Bunding (ha), Graded Bunding (ha), Other Watershed Activities (Trench cum Bund etc) (ha) |
| Intensive Horticulture / Afforestation | Area Brought Under Plantation (Horticulture/Afforestation) (ha), Others (Intensive Afforestation - Nurseries etc) (in lakhs) |
| *(ungrouped, single column, rowSpan)* | Awareness Programme (Jatha, Beedi Nataka, Grama Sabha, Trainings, Kisan Melas etc) (Nos) |
| Springshed Rejuvenation | Activity Related to Springshed Rejuvenation (Total No. of Activities Taken), Number of Springs Rejuvenated (Nos) |

24 data columns total (2 leading + 22 numeric), matching the source PDF's 26 numbered columns (columns 1–2 are Sr No/District, handled as row grouping rather than table columns here).

**R1 has real, verified data** for all 30 districts for period "March 2026" — every project/block name and every numeric figure was extracted directly from the source PDF (`uploads/pmksy_report.txt` in the design project, or re-derive from the original PDF). Seed your database with this real data for R1/March 2026 rather than placeholder numbers.

## Districts (all 30, real)
Bagalkot, Bengaluru, Belagavi, Ballari, Vijayanagar, Bidar, Vijayapura, Chamarajanagar, Chikkaballapura, Chikkamagaluru, Chitradurga, Dakshina Kannada, Davangere, Dharwad, Gadag, Kalburgi, Hassan, Haveri, Kodagu, Kolar, Koppal, Mandya, Mysuru, Raichur, Ramanagara, Shivamogga, Tumkur, Udupi, Uttara Kannada, Yadgir.

## Interactions & Behavior
- **Status lifecycle** per (district, format, period): `not_started` → `draft` (first entry added, or explicit Save Draft) → `submitted` (Submit Report; locks further district edits). Admin can lock/reopen an entire period (blocks all district edits regardless of submission status while locked).
- **Entry CRUD**: Add (inline form matching the format's columns) → appears in the table; Edit (loads entry back into the inline form, "Update Entry" replaces it in place); Remove (deletes immediately, no confirmation in the prototype — consider adding one in production).
- **Toasts**: bottom-right, auto-dismiss ~2.5s, used for Save Draft, Submit, period create/lock/reopen, and all export actions.
- **Pie charts**: recompute live from current submission entry counts; update immediately if data changes (e.g. after an entry is added, if viewed live).
- **Export buttons** (district "Export All" + per-format, admin per-format + per-consolidated-report): currently just show a toast — must be wired to real file generation in production.

## Design Tokens
- **Font**: Public Sans (400/500/600/700/800), loaded from Google Fonts.
- **Colors** (all OKLCH):
  - Primary/header navy: `oklch(24% 0.055 250)` (darker variant `oklch(28% 0.06 250)` for buttons/accents)
  - App background: `oklch(97% 0.006 95)`; card surface: `white`; borders: `oklch(88% 0.01 95)` / `oklch(85% 0.01 95)`
  - Text: primary `oklch(22% 0.01 95)`, muted `oklch(48%–55% 0.01 95)`
  - Status — submitted: bg `oklch(90% 0.09 150)` / fg `oklch(32% 0.1 150)`; draft: bg `oklch(94% 0.09 85)` / fg `oklch(38% 0.12 75)`; not started: bg `oklch(94% 0.005 95)` / fg `oklch(50% 0.01 95)`
  - Pie chart slice hues: evenly distributed around the hue wheel at `oklch(64% 0.11 <hue>)`, hue = `i * 360 / n`
- **Radii**: 6–10px on cards/buttons/inputs, 20px (pill) on status chips and period pills.
- **Spacing**: card padding 18–20px, section gaps 16–22px, grid `auto-fill`/`auto-fit` with `minmax(300–360px, 1fr)`.

## Assets
No images/icons — a circular "KA" initials mark (Karnataka) is a plain styled `div`, not an image asset. No SVG illustrations used.

## Files
- `PMKSY Reporting Portal.dc.html` — the full working prototype (open directly in a browser; it's self-contained aside from a `support.js` runtime file used only by the prototyping tool, and a Google Fonts link).
- `pmksy_source_report.txt` — plain-text extraction of the original PDF (all 11 tables' raw text), useful for re-verifying or extracting the remaining 10 formats' real column structures/data.
