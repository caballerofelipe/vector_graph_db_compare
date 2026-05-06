# DB Comparison Table — LLM context

Dense notes for assistants working in this repo. For how to run the page and what it does for users, see `README.md`. A human-oriented companion piece lives off-repo as the Substack **article** at `https://sometech.substack.com/p/a-comparison-table-for-vector-and` (rationale and commentary only — this file is the source of truth for implementation and data rules).

**Page title** (in `db_comparison_table.html`): “Vector **and/or** Graph DB …” — signals that listed products are not required to be both vector and graph engines.

## Stack

- `db_comparison_table.html` loads `db_comparison_data.js` (same directory). No build step. Footer: **article** (Substack companion — URL in opening paragraph), **contact** (GitHub issues).
- View state persists under `localStorage` key **`dbcmp`** as JSON: **`q`**, **`filters`** (pill map), **`cols`** (visibility map). Theme is not persisted.
- Caveat copy lives in the CSV **`notes`** column (JSON keyed by boolean field ids). Every boolean column uses **`boolCell(v, note)`** (see **Boolean caveat renderers**): **on** + `note` → orange yes-dot; **on** without `note` → green; **off** + `note` → orange-ring gray off-dot; **off** without `note` → plain gray dot.
- Feature pills for a column are hidden when that column is hidden; their filters are cleared.

## UI behavior (implementation)

- **Theme** — `themeMode` cycles **system → light → dark → system** (`nextMode` in script). Default is system (`prefers-color-scheme`). `body.light` toggles light palette. **Not** stored in `localStorage` (resets on reload).
- **Feature filter pills** — Each boolean column’s filter is `null | true | false`. Click cycles **`null` → `true` (require on, green `.on`) → `false` (exclude off, pink `.off`) → `null`**. `true` keeps rows with `r[key] === 1`; `false` keeps rows with `r[key] === 0`.
- **Text search** — Case-insensitive substring over **`name` + `tagline` + `ideal` + `released` + `langs` + `implang` + `compat` + string form of `ghstars` (when present)** only (not URL/source).
- **Sort** — `sortCol` + `sortDir` (`1` or `-1`). Clicking the active header flips direction; a new column starts ascending. Boolean fields (and other numeric `0/1` columns) sort numerically; **`ghstars`** sorts numerically when both sides have counts, with **`null`** (no **`ghstars`** cell) sorted **after** all numbers ascending and **before** all numbers descending. All other columns use `localeCompare` on strings (**`url`** and **`src`** included — empty sorts as empty string). **`th.onclick`** resets every **`.arr`** to **↕** then sets **↑ / ↓** on the active header only if **`querySelector('.arr')`** exists (defensive).
- **Tooltips** — One fixed `#tooltip` div; **document-level** `mouseover` / `mousemove` / `mouseout` with `e.target.closest('[data-tip]')`. Copy comes from `dataset.tip` (table headers use `th[data-tip]`; header tips are scraped into `colTips` for filter pills). **Text columns** (**`tagline`**, **`ideal`**, **`released`**, **`langs`**, **`implang`**, **`compat`**, **`ghstars`**) set **`data-tip`** on **`td`**s with the **full cell text** via **`cellTip`** / **`dataTipEscaped`** (omit when empty). **Position:** **`positionTip`** uses **`offsetWidth` / `offsetHeight`** after text is applied: default southeast of cursor (`clientX/Y` + **`pad`**); flips northwest when that would clip the viewport, then clamps to margins (avoids a fixed guessed width separating the tooltip from the pointer).
- **Column visibility** — `colGroups` labels: INFO, TYPE, DEPLOYMENT, STORAGE, CAPABILITIES, INTEGRATIONS, LINKS (see script for key lists). **`INFO`** uses **`['tagline','ideal','released','active']`** (**Ideal use** follows **Tagline** in the table and CSV). **`INTEGRATIONS`** uses **`['lc','langs','implang','compat']`**; **`LINKS`** uses **`['url','src','ghstars']`** — **LC/LI** appears before **Languages** and **implementation language (`implang`)** in both the table header/body and the CSV field order (after **Cypher**). `#tbl` **`minWidth`** = `NAME_WIDTH` (120) + sum of `colWidths` for visible columns; hidden columns get `display:none` via injected `#colStyle`. **All** / **None** / per-pill toggles; hiding a column clears its filter. **Reset** clears search, all filters, and restores `colDefaultVisible`.
- **Stats row** — `#statsCount` shows filtered row count (“Showing *N* of *M* databases”). Static **`.stats-legend`** on the same row (flex; `margin-left: auto` on the legend) lists the four **`boolCell`** dot styles: green yes, orange yes (caveat), plain gray no, gray no + orange ring (off caveat). The legend does **not** repeat caveat wording or long text-cell copy; **`data-tip`** on boolean caveat markers and on **text** **`td`**s (see **Tooltips**) holds full text.
- **CSV export** — `↓ CSV` builds a Blob from **`window.DB_CSV_DATA`**; download name **`db_comparison.csv`**.
- **Parsing** — `BOOL_KEYS` in script lists fields coerced to `0/1`; **`ghstars`** is coerced to **`number \| null`** (empty cell → null); `notes` is `JSON.parse` when non-empty. Adding a **boolean** column requires updating **`BOOL_KEYS`**, **`boolCols`** (filter pills), thead `th`, row template (see **Boolean caveat renderers**), **`colGroups`** / **`colLabels`** / **`colWidths`** (and **`colDefaultVisible`** / search if needed). Adding a **free-text** column requires header + body **`td`**, CSV key in **`db_comparison_data.js`**, the same **`col*`** maps, and (if the column should show **`#tooltip`** on hover) **`cellTip(...)`** wrapping in the row template.

### Boolean caveat renderers (`boolCell`)

**`boolCell(v, note)`** in `db_comparison_table.html` renders every boolean column (`v` is `0`/`1` after parse; `note` is `r.notes?.<key>`).

| `v` | `note` | Markup |
|-----|--------|--------|
| `1` | absent / empty | Green `.yes-dot` |
| `1` | present | Orange `.yes-dot.note-dot` + `data-tip` (escape `"` in `note` for `dataset.tip`) |
| `0` | absent / empty | Plain `.no-dot` |
| `0` | present | **Inactive-note off-dot** `<span class="no-dot inactive-note" data-tip="…">` |

**Inactive-note off-dot:** gray **no** marker plus orange ring (`.inactive-note`) so it reads as “off — see tooltip for why.” Same **`data-tip`** + document-level tooltip handler as on-caveats. In light theme, keep **`.inactive-note`** in the same “no glow” rule as **`.note-dot`**.

**Data:** Put caveat copy in **`notes.<key>`** for that boolean. Omit the key (or use an empty string) when **off** needs no explanation — do **not** leave stray keys for `0` cells unless you intend an off-caveat.

## Dataset

- **Source of truth:** In `db_comparison_data.js`, each data line of `window.DB_CSV_DATA` (the multiline string at the top of the file) is one product row, excluding the CSV header line. The stats row’s “*M* databases” total is computed at runtime from that file — **do not keep a manual row count in this document**; it will drift.

- **Row order:** Those data rows (not the header) are kept **sorted alphabetically by `name`**, case-insensitive (compare `name.toLowerCase()` with string `<` / `>`). Preserve that order when adding or renaming rows so the file stays aligned with `README.md` (product list under **Scope and limitations**).

- **Deliberately omitted products:** **Turbopuffer** and **ClickHouse** were considered and left out (analytics-first / narrower fit for this grid than the rest). **Cognee** ([https://github.com/topoteretes/cognee](https://github.com/topoteretes/cognee)) is intentionally omitted: it is an agent **memory control plane** (ingestion, pipelines, recall APIs) that uses embeddings and graph-style structure, not a first-class vector or graph **database** product comparable row-for-row with the stores in the table.

- **Caveats:** There is **no** separate caveat inventory file. Caveat text exists only in each row’s **`notes`** JSON in `db_comparison_data.js`.

- **Editing `db_comparison_data.js`:** The CSV is embedded in a JavaScript **template literal** (grave-accent delimiters). A raw grave accent inside the CSV body ends the literal early and **breaks the page load**. Do not wrap paths or commands in Markdown-style code ticks inside the CSV text; use plain text. If you truly need that character inside the string, use the normal JavaScript template-literal escape (backslash immediately before the grave accent).

## CSV schema (`db_comparison_data.js`)

Single export: `window.DB_CSV_DATA` (multi-line CSV string). Edit this file only for row/cell changes.

**Header (field order):**

`name,tagline,ideal,released,active,vector,graph,oss,selfhost,lightweight,docker,single,lowops,ram,persistence,server,embeddable,traversal,hybrid,rag,cypher,lc,langs,implang,compat,url,src,ghstars,notes` (matches the embedded string in `db_comparison_data.js`; no spaces after commas)

**Rules:** Booleans: `1` / `0`. Strings: plain text; double-quote fields that contain commas. `notes`: JSON object for caveat tooltips, CSV-escaped (`""` for quotes inside); empty if none.

**UI label → key** (short headers in the table map to these ids):

| Key | Typical UI |
|-----|------------|
| `name` | DB Name |
| `active` | Status (actively maintained vs discontinued / archived / effectively unmaintained) |
| `vector`, `graph` | Vector DB, Graph DB |
| `oss`, `selfhost`, `lightweight`, `docker`, `single`, `lowops` | OSS, Self-hosted, … |
| `ram`, `persistence`, `server`, `embeddable` | RAM, Persistence, Server, Embeddable |
| `traversal`, `hybrid`, `rag`, `cypher` | Traversal, Hybrid, RAG, Cypher |
| `lc`, `langs`, `implang`, `compat` | LC/LI, Languages, Impl. lang, OS Compat |
| `url`, `src`, `ghstars` | URL, Source, GH stars |
| `released` | Released (`YYYY-MM`) |
| `tagline` | Tagline |
| `ideal` | Ideal use |
| `notes` | Not shown as a column; JSON keys match boolean field ids — a key may supply a caveat for **on** or **off** depending on the cell value (see **`boolCell`**) |

### Boolean keys — meaning for `1` / `0`

Use these definitions when filling or auditing cells (judgment calls, not vendor specs):

| Key | `1` means |
|-----|-----------|
| `active` | **On:** default-branch GitHub activity within the **six months before the last table check (2026-05-04)** or a clearly ongoing commercial/cloud product without a single OSS repo to measure — use **`notes.active`** when the cell stays on but the repo is stale (yes-caveat). **Off:** GitHub repo **archived/read-only**, formal discontinuation, or similar — set **`notes.active`** for an off-caveat via **`boolCell`**. |
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

**Release / status audit stamp (2026-05-04):** see **Released** and **Status** header tooltips (`th[data-col="released"]`, `th[data-col="active"]`).

**Other string columns:** `name` — product label; `tagline` — one-line positioning; `ideal` — concise ideal use case / fit (editorial: where the product shines vs generic marketing); `released` — approximate first public availability **`YYYY-MM`**; `langs` — official client SDKs (free text); `implang` — primary implementation language(s) of the core server/engine (free text, from public source or stated stack; use **Proprietary (undisclosed)** when not public); `compat` — OS support without Docker as the assumed path; `url` / `src` — website and repo URLs (empty if none); **`ghstars`** — GitHub **`stargazers_count`** on the repo in **`src`** (ASCII integer; empty when no **`src`** or not applicable; refresh periodically — last filled **2026-05-06** in this repo).

### New boolean columns

- Render every boolean **`<td>`** with **`boolCell(r.<key>, r.notes?.<key>)`**. Caveats on **off** use the same **`notes.<key>`** mechanism as on — only include text when you want the orange-ring off-dot.
- Keep **`th[data-tip]`** aligned with what that column measures (not with a separate “off mode” flag; there is none).

## UI tokens (when editing HTML/CSS)

- Dark bg default `#0d0f14`; accent yes `#4ade80`; caveat yes `#f97316`.
- Fonts: Space Mono (headers/mono), DM Sans (body); loaded from Google Fonts.
- Boolean columns: width **55px** (`.col-bool`). **Released** matches width **55px** (`.col-released`) but uses **`Space Mono`** + **tabular nums** + **11px** (with **`.langs-cell`**); **GH ★** is the same mono/tabular styling, right-aligned (**`.col-stars`**). **Tagline** and **Ideal use** share width **175px** (**`.col-tag`**, **`.col-ideal`**). Glow on yes-dots off in light theme.
- **Off-with-caveat:** `.no-dot.inactive-note` — orange border / shadow on the gray off dot; pair with `data-tip` (see **Boolean caveat renderers**).
- **Default visible** toggleable columns: Tagline, Ideal use, Released, Status, Vector DB, Graph DB, OSS, Self-hosted, Persistence, Server, OS Compat, URL, Source, GH stars (14). DB Name always on; rest via COLUMNS panel.

## Known gaps

- No mobile-first layout; expect horizontal scroll on narrow viewports.

