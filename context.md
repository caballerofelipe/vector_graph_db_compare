# DB Comparison Table — LLM context

Dense notes for assistants working in this repo. For how to run the page and what it does for users, see `README.md`.

## Stack

- `db_comparison_table.html` loads `db_comparison_data.js` (same directory). No build step.
- View state persists under `localStorage` key **`dbcmp`** as JSON: **`q`**, **`filters`** (pill map), **`cols`** (visibility map). Theme is not persisted.
- Caveat copy lives in the CSV **`notes`** column (JSON keyed by boolean field ids). Orange dots only on “on” cells that have a note; “off” cells never show a note marker.
- Feature pills for a column are hidden when that column is hidden; their filters are cleared.

## UI behavior (implementation)

- **Theme** — `themeMode` cycles **system → light → dark → system** (`nextMode` in script). Default is system (`prefers-color-scheme`). `body.light` toggles light palette. **Not** stored in `localStorage` (resets on reload).
- **Feature filter pills** — Each boolean column’s filter is `null | true | false`. Click cycles **`null` → `true` (require on, green `.on`) → `false` (exclude off, pink `.off`) → `null`**. `true` keeps rows with `r[key] === 1`; `false` keeps rows with `r[key] === 0`.
- **Text search** — Case-insensitive substring over **`name` + `tagline` + `released` + `langs` + `compat`** only (not URL/source).
- **Sort** — `sortCol` + `sortDir` (`1` or `-1`). Clicking the active header flips direction; a new column starts ascending. Numbers sort numerically; everything else uses `localeCompare` on strings.
- **Tooltips** — One fixed `#tooltip` div; **document-level** `mouseover` / `mousemove` / `mouseout` with `e.target.closest('[data-tip]')`. Copy comes from `dataset.tip` (table headers use `th[data-tip]`; header tips are scraped into `colTips` for filter pills). **Position:** cursor + 14px; if `x + 240 > innerWidth` then `x = clientX - 240`; if `y + 120 > innerHeight` then `y = clientY - 90` — coarse, hence awkward near edges.
- **Column visibility** — `colGroups` labels: INFO, TYPE, DEPLOYMENT, STORAGE, CAPABILITIES, INTEGRATIONS, LINKS (see script for key lists). `#tbl` **`minWidth`** = `NAME_WIDTH` (120) + sum of `colWidths` for visible columns; hidden columns get `display:none` via injected `#colStyle`. **All** / **None** / per-pill toggles; hiding a column clears its filter. **Reset** clears search, all filters, and restores `colDefaultVisible`.
- **CSV export** — `↓ CSV` builds a Blob from **`window.DB_CSV_DATA`**; download name **`db_comparison.csv`**.
- **Parsing** — `BOOL_KEYS` in script lists fields coerced to `0/1`; `notes` is `JSON.parse` when non-empty. Adding a boolean column requires updating **`BOOL_KEYS`**, **`boolCols`** (filter pills), thead `th`, row template `bc(...)`, **`colGroups`** / **`colLabels`** / **`colWidths`** (and defaults if needed).

## Dataset

25 databases in the CSV. **Kuzu** excluded by project choice. **Vespa** and **SurrealDB** were added to the original list. **Turbopuffer** and **ClickHouse** were considered and left out here (analytics-first / narrower fit for this grid than the rest). There is **no** separate caveat inventory in the repo — caveat text exists only in each row’s **`notes`** JSON in `db_comparison_data.js`.

## CSV schema (`db_comparison_data.js`)

Single export: `window.DB_CSV_DATA` (multi-line CSV string). Edit this file only for row/cell changes.

**Header (field order):**

`name, tagline, released, vector, graph, oss, selfhost, lightweight, docker, single, lowops, ram, persistence, server, embeddable, traversal, hybrid, rag, cypher, langs, lc, compat, url, src, notes`

**Rules:** Booleans: `1` / `0`. Strings: plain text; double-quote fields that contain commas. `notes`: JSON object for caveat tooltips, CSV-escaped (`""` for quotes inside); empty if none.

**UI label → key** (short headers in the table map to these ids):

| Key | Typical UI |
|-----|------------|
| `name` | DB Name |
| `vector`, `graph` | Vector DB, Graph DB |
| `oss`, `selfhost`, `lightweight`, `docker`, `single`, `lowops` | OSS, Self-hosted, … |
| `ram`, `persistence`, `server`, `embeddable` | RAM, Persistence, Server, Embeddable |
| `traversal`, `hybrid`, `rag`, `cypher` | Traversal, Hybrid, RAG, Cypher |
| `langs`, `lc`, `compat` | Languages, LC/LI, OS Compat |
| `url`, `src` | URL, Source |
| `released` | Released (`YYYY-MM`) |
| `tagline` | Tagline |
| `notes` | Not shown as a column; drives caveat tooltips on boolean cells |

### Boolean keys — meaning for `1` / `0`

Use these definitions when filling or auditing cells (judgment calls, not vendor specs):

| Key | `1` means |
|-----|-----------|
| `vector` | Product supports storing/querying high-dimensional embeddings (vector workload). |
| `graph` | Graph model: nodes/edges and relationship-centric queries. |
| `oss` | Source publicly available under an **OSI-approved** license (project’s reading for each product). |
| `selfhost` | Can run on infrastructure you control (not SaaS-only). |
| `lightweight` | Viable on modest CPU/RAM (laptop/small server/edge). |
| `docker` | Official Docker image exists. |
| `single` | Usable on one machine without a multi-node cluster requirement. |
| `lowops` | Low day-2 operational burden for a typical deployment. |
| `ram` | Workload/design is **RAM-first** (even if disk exists). |
| `persistence` | Durable across restarts under normal configuration (not pure ephemeral-only). |
| `server` | Can run as a network-accessible server process. |
| `embeddable` | In-process / library-style embedding without a mandatory separate server. |
| `traversal` | Multi-hop graph traversal / path-style queries supported. |
| `hybrid` | Single-query (or first-class) **vector + sparse/keyword** style retrieval. |
| `rag` | Reasonable fit as a **retrieval** layer in LLM/RAG stacks. |
| `cypher` | Cypher (or marketed as Cypher-compatible) graph query support. |
| `lc` | Native LangChain and/or LlamaIndex integration. |

**Other string columns:** `name` — product label; `tagline` — one-line positioning; `released` — approximate first public availability **`YYYY-MM`**; `langs` — official client SDKs (free text); `compat` — OS support without Docker as the assumed path; `url` / `src` — website and repo URLs (empty if none).

### New boolean columns

Every `BOOL_KEYS` field rendered with **`bc(r.<key>, r.notes?.<key>)`** so `notes.<key>` can show an orange caveat dot when the cell is `1`. If you add a column, keep that pairing in the row template.

## UI tokens (when editing HTML/CSS)

- Dark bg default `#0d0f14`; accent yes `#4ade80`; caveat yes `#f97316`.
- Fonts: Space Mono (headers/mono), DM Sans (body); loaded from Google Fonts.
- Boolean and released columns: width **55px** (`.col-bool`, `.col-released`). Glow on yes-dots off in light theme.
- **Default visible** toggleable columns: Tagline, Released, Vector DB, Graph DB, OSS, Self-hosted, Persistence, Server, OS Compat, URL, Source (11). DB Name always on; rest via COLUMNS panel.

## Known gaps

- Tooltip positioning near viewport edges is rough.
- No mobile layout; wide table + horizontal scroll.
