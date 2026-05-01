# DB Comparison Table — Session Context

## What was built

A self-contained, interactive HTML file (`db_comparison_table.html`) comparing 25 databases (vector, graph, and multi-model) across 20 columns. It has:

- **Sortable columns** — click any header to sort ascending/descending
- **Three-state feature filter pills** — click to cycle: none (no filter) → on (green, require feature) → off (pink, exclude feature)
- **Text search** — filters by name, tagline, language, or OS
- **Hover tooltips** on every column header explaining what the column means (implemented via a JS-driven fixed-position `div`, not CSS `::after`, because sticky headers clip pseudo-element tooltips)

---

## Databases included (25 total)

Original list provided by user, plus two additions recommended by Claude:

| Added | Reason |
|---|---|
| **Vespa** | Yahoo's open-source enterprise search + vector platform; frequently in top-10 lists for billion-scale hybrid search |
| **SurrealDB** | Genuinely multi-model (graph, doc, vector, SQL) in a single lightweight binary; strong LangChain support |

Excluded per user instruction: **Kuzu**

Turbopuffer and ClickHouse were considered but not added (too niche / primarily analytics).

---

## Columns defined

| Column | Key | Description |
|---|---|---|
| DB Name | `name` | Name of the database or combination |
| Tagline | `tagline` | Short description of primary purpose |
| Vec | `vector` | Vector DB — stores/queries high-dimensional embeddings |
| Graph | `graph` | Graph DB — nodes + edges, relationship traversal |
| OSS | `oss` | Open source under an OSI-approved license |
| Self-H | `selfhost` | Can be deployed on your own infrastructure |
| Light | `lightweight` | Runs with minimal CPU/RAM; edge/laptop-friendly |
| Dock | `docker` | Official Docker image available |
| 1-Node | `single` | Runs on a single machine, no cluster needed |
| Low-Ops | `lowops` | Minimal ongoing maintenance overhead |
| Embed | `embeddable` | **Embedded in the SQLite sense** — runs in-process, stores to local files, no server process needed |
| Trav | `traversal` | Graph traversal — multi-hop queries, path-finding |
| Hybrid | `hybrid` | Hybrid search — vector + keyword/BM25 in one query |
| RAG | `rag` | Integrates well as retrieval layer in RAG pipelines |
| Cypher | `cypher` | Supports Cypher graph query language |
| Languages | `langs` | Official client SDKs available |
| LC/LI | `lc` | LangChain or LlamaIndex native integration |
| OS Compat | `compat` | Operating systems supported natively |
| URL | `url` | Official website |
| Source | `src` | GitHub / source code repository |

---

## Design

- Dark theme (`#0d0f14` background)
- Fonts: `Space Mono` (headers/mono), `DM Sans` (body)
- Green accent (`#4ade80`) for active states and yes-dots
- Boolean values shown as green glowing dot (✓) or dark empty dot (–)
- Fully self-contained — no external runtime dependencies (fonts load from Google Fonts)

---

## Known issues / things to revisit

- The incomplete `thead` block from a failed mid-session edit was cleaned up; the final file is valid
- Tooltip positioning flips left/up near screen edges
- No mobile layout — table requires horizontal scroll on small screens

---

## Pending changes (not yet implemented)

### UI / UX
- **Change title** to "Vector and Graph DB Comparison Table"
- **Column visibility toggle** — allow turning individual columns on/off so the user can hide columns they don't care about
- **Tooltip on filter pills** — the filter buttons should show the same definition tooltip as the corresponding column header

### Cell-level notes
- Some cells need nuance beyond a simple yes/no dot. If a cell has a note, display a small **ⓘ** (i in a circle) next to the dot. Hovering it shows the note as a tooltip (same JS-driven tooltip mechanism already in place).
- Examples of cells that need notes:
  - Chroma → Persistence: "Persistent mode only; default in-memory mode has no persistence"
  - CozoDB → Embeddable: "Backend is selectable: in-memory, SQLite, or RocksDB"
  - CozoDB → Persistence: "Depends on backend; RocksDB/SQLite backends are durable"
  - FAISS → Persistence: "No built-in persistence; caller must manually save/load index files"
  - FAISS → Server: "Library only — no server mode; must be wrapped by the application"
  - FalkorDB → RAM: "Built on Redis; data is RAM-first with optional persistence"
  - Memgraph → Embeddable: "In-memory server; not embeddable in the SQLite sense"
  - SurrealDB → Embeddable: "Embedded mode available via the embedded feature flag"
  - Pinecone → Self-hosted: "SaaS only — no self-host option"
  - MongoDB Atlas Vector → Self-hosted: "Atlas is cloud-only; self-hosted MongoDB requires separate vector setup"
- The ⓘ icon should be subtle — small, muted color, only visible on row hover or always visible but unobtrusive

### Light theme
- Add a **light/dark theme toggle** button in the header area
- Default can be dark (current) or follow system `prefers-color-scheme`
- Light theme: white/off-white backgrounds, dark text, same green accent (slightly deeper for contrast on light)
- All colors must use CSS variables — no hardcoded hex in component styles — so the theme swap is a single class toggle on `<body>` or `:root`
- Theme preference persisted in `localStorage`

### CSV-driven data
- Extract all DB records out of the HTML into a separate **`db_comparison.csv`** file
- Both files live in the same directory; HTML loads the CSV via `fetch()` on page load with a relative path
- No DB data should be hardcoded in the JS — everything comes from the CSV
- **CSV format:**
  - Header row uses the JS data keys: `name, tagline, vector, graph, oss, selfhost, lightweight, docker, single, lowops, embeddable, traversal, hybrid, rag, cypher, langs, lc, compat, url, src, ram, persistence, server, notes` (note: `embeddable` column replaces the old `filebased`)
  - Boolean columns: `1` = yes, `0` = no
  - String columns (`langs`, `compat`, `url`, `src`, `tagline`): plain text, double-quoted if they contain commas
  - **`notes` column**: JSON-encoded object keyed by column name, double-quoted and CSV-escaped. Example: `"{""embeddable"":""Embedded mode available; server mode also supported"",""ram"":""RAM-first by design""}"`
  - Empty notes: leave the cell empty or `{}`
  - Add a link to allow the user to download the CSV directly for their usage.

### Cell-level notes (ⓘ icon)
- If a DB's `notes` object has an entry for a given column, render a small **ⓘ** icon inside that cell next to the dot
- Hovering ⓘ shows the note text using the same JS-driven fixed-position tooltip already in place
- The ⓘ should be subtle — small, muted color, always visible (not just on hover)
- Cells with no note show only the dot, no icon
- Known notes to include (minimum):

| DB | Column | Note |
|---|---|---|
| FAISS | persistence | No built-in persistence; caller must manually save/load index files |
| FAISS | server | Library only — no server mode; must be wrapped by the application |
| Chroma | embeddable | Embedded mode available; persistent-client and server modes also supported |
| Chroma | persistence | Default in-memory mode has no persistence; use PersistentClient for durability |
| CozoDB | embeddable | Backend is configurable: in-memory, SQLite, or RocksDB |
| CozoDB | persistence | Depends on backend; SQLite and RocksDB backends are durable |
| FalkorDB | ram | Built on Redis; data is RAM-first with optional AOF/RDB persistence |
| Memgraph | embeddable | In-memory server; not embeddable in the SQLite sense — requires a server process |
| SurrealDB | embeddable | Embedded mode available via the embedded feature flag in Rust/Python |
| Pinecone | selfhost | SaaS only — no self-host option |
| MongoDB Atlas Vector | selfhost | Atlas is cloud-only; self-hosted MongoDB requires separate vector configuration |

### New columns to add

| Column | Key | Description |
|---|---|---|
| RAM | `ram` | Runs primarily in RAM — data held in memory for fast access (e.g. Redis, Memgraph, FalkorDB) |
| Persistence | `persistence` | Saves state to disk so data survives restarts — no data loss on reboot |
| Server | `server` | Can be run as a standalone server process accessible over a network |

### Data to fill in for new columns (research needed)
All values below are a starting point — verify before shipping:

| DB | RAM | Persistence | Server |
|---|---|---|---|
| Qdrant | 0 | 1 | 1 |
| Chroma | 0 | 1 | 1 |
| Weaviate | 0 | 1 | 1 |
| Milvus | 0 | 1 | 1 |
| Pinecone | 0 | 1 | 1 |
| LanceDB | 0 | 1 | 1 |
| pgvector | 0 | 1 | 1 |
| pgvectorscale | 0 | 1 | 1 |
| FAISS | 0 | 0 | 0 |
| Elasticsearch | 0 | 1 | 1 |
| MongoDB Atlas Vector | 0 | 1 | 1 |
| Redis Vector | 1 | 1 | 1 |
| Neo4j | 0 | 1 | 1 |
| Memgraph | 1 | 1 | 1 |
| TigerGraph | 0 | 1 | 1 |
| Amazon Neptune | 0 | 1 | 1 |
| FalkorDB | 1 | 1 | 1 |
| JanusGraph | 0 | 1 | 1 |
| OrientDB | 0 | 1 | 1 |
| ArcadeDB | 0 | 1 | 1 |
| ArangoDB | 0 | 1 | 1 |
| PG + pgvector + AGE | 0 | 1 | 1 |
| CozoDB | 0 | 1 | 1 |
| Vespa | 0 | 1 | 1 |
| SurrealDB | 0 | 1 | 1 |

Notes:
- **FalkorDB** runs on top of Redis so it is RAM-first with optional persistence
- **FAISS** is a library — it has no built-in persistence or server mode; the caller is responsible for saving/loading index files
- **Chroma** persistence depends on mode: in-memory (no persistence) or persistent client (yes); mark as 1 since persistent mode is the common production path
- **CozoDB** persistence depends on chosen backend (in-memory, SQLite, RocksDB); mark as 1 since durable backends are default for production

---

## Files delivered

- `db_comparison_table.html` — single self-contained file, open in any browser
- `context.md` — this file
