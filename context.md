# DB Comparison Table — Session Context

## What was built

A self-contained, interactive HTML file (`db_comparison_table.html`) comparing 25 databases (vector, graph, and multi-model) across 23 columns. It has:

- **Sortable columns** — click any header to sort ascending/descending
- **Three-state feature filter pills** — click to cycle: none (no filter) → on (green, require feature) → off (pink, exclude feature)
- **Text search** — filters by name, tagline, language, or OS
- **Hover tooltips** on every column header and every filter pill (JS-driven fixed-position `div` with document-level event delegation)
- **Cell-level caveat notes** — boolean cells with a nuance show an orange dot instead of green; hovering reveals the note. Only "on" cells with genuine caveats get this treatment; "off" cells never show a note indicator.
- **Light/dark theme toggle** — three-state (System / Light / Dark), defaults to system `prefers-color-scheme`

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
| RAM | `ram` | Data held primarily in memory for fast access |
| Persist | `persistence` | Saves state to disk; data survives restarts |
| Server | `server` | Can run as a standalone server process over a network |
| Embed | `embeddable` | Runs in-process (SQLite-style) — no server process needed |
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

## Cell-level caveat notes (orange dots)

A green "on" dot turns orange when the feature has a caveat worth knowing. Hovering shows the note. Current orange dots:

| DB | Column | Note |
|---|---|---|
| Weaviate | OSS | BSL 1.1 from v1.25+ — not OSI-approved; converts to Apache 2.0 after 4 years |
| FAISS | Embeddable | In-process library with no storage engine; caller is fully responsible for saving and loading index files |
| Chroma | Embeddable | Embedded mode available; persistent-client and server modes also supported |
| Chroma | Persistence | Default in-memory mode has no persistence; use PersistentClient for durability |
| Redis Vector | RAM | All data lives in RAM; disk persistence must be explicitly configured via AOF or RDB — not enabled by default |
| Redis Vector | Persistence | Persistence via AOF or RDB snapshots; must be explicitly enabled in redis.conf — off by default |
| Memgraph | Persistence | Persists via periodic snapshots and WAL; primarily in-memory — recent in-flight writes may be lost on unexpected crash |
| FalkorDB | RAM | Built on Redis; data is RAM-first with optional AOF/RDB persistence |
| FalkorDB | Persistence | Relies on Redis AOF/RDB persistence; must be explicitly enabled in Redis config — not on by default |
| JanusGraph | Docker | Official image requires an external storage backend (Cassandra, HBase, or BerkeleyDB) — additional services needed alongside it |
| ArangoDB | OSS | Licensed under BSL 1.1 from v3.11+ — not OSI-approved open source; converts to Apache 2.0 after 4 years |
| PG + pgvector + AGE | Cypher | Cypher support via Apache AGE — implementation is partial; not all Cypher clauses or functions are supported |
| CozoDB | Embeddable | Backend is configurable: in-memory, SQLite, or RocksDB |
| CozoDB | Persistence | Depends on backend; SQLite and RocksDB backends are durable |
| SurrealDB | Embeddable | Embedded mode available via the embedded feature flag in Rust/Python |

---

## Design

- Dark theme default (`#0d0f14` background), light theme available
- Fonts: `Space Mono` (headers/mono), `DM Sans` (body)
- Green accent (`#4ade80`) for active states and yes-dots
- Orange dot (`#f97316`) for yes-with-caveat cells
- Boolean values shown as green/orange glowing dot (yes) or dark empty dot (no)
- Fully self-contained — no external runtime dependencies (fonts load from Google Fonts)

---

## Known issues / things to revisit

- Tooltip positioning flips left/up near screen edges
- No mobile layout — table requires horizontal scroll on small screens

---

## Pending changes (not yet implemented)

### UI / UX
- **Column visibility toggle** — allow turning individual columns on/off so the user can hide columns they don't care about

### CSV-driven data
- Extract all DB records out of the HTML into a separate **`db_comparison.csv`** file
- Both files live in the same directory; HTML loads the CSV via `fetch()` on page load with a relative path
- No DB data should be hardcoded in the JS — everything comes from the CSV
- **CSV format:**
  - Header row uses the JS data keys: `name, tagline, vector, graph, oss, selfhost, lightweight, docker, single, lowops, ram, persistence, server, embeddable, traversal, hybrid, rag, cypher, langs, lc, compat, url, src, notes`
  - Boolean columns: `1` = yes, `0` = no
  - String columns (`langs`, `compat`, `url`, `src`, `tagline`): plain text, double-quoted if they contain commas
  - **`notes` column**: JSON-encoded object keyed by column name, double-quoted and CSV-escaped. Example: `"{""embeddable"":""Embedded mode available; server mode also supported"",""ram"":""RAM-first by design""}"`
  - Empty notes: leave the cell empty or `{}`
  - Add a link to allow the user to download the CSV directly for their usage

---

## Files delivered

- `db_comparison_table.html` — single self-contained file, open in any browser
- `context.md` — this file
