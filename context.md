# DB Comparison Table — Session Context

## What was built

A self-contained, interactive HTML file (`db_comparison_table.html`) comparing 25 databases (vector, graph, and multi-model) across 20 columns. It has:

- **Sortable columns** — click any header to sort ascending/descending
- **Feature filter pills** — click to require a feature; only matching DBs show
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
| File | `filebased` | **Embedded/file-based in the SQLite sense** — runs in-process, stores to local files, no server process needed |
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

## Important clarification: "File-based" column

The user clarified mid-session that **file-based means SQLite-style embedded storage** — the DB runs in-process with no separate server, storing data directly in local files. It does NOT mean "has disk persistence" (which almost all DBs have).

### Databases marked file-based ✓
- **Chroma** — embedded mode with local file storage
- **LanceDB** — core design is file-based (Lance columnar format)
- **FAISS** — saves/loads index files manually (library, not a server)
- **OrientDB** — stores data in local files
- **ArcadeDB** — file-based local storage
- **CozoDB** — embeddable with RocksDB or SQLite backend
- **SurrealDB** — embedded mode with file storage

### Notable NOT file-based
- **Memgraph** — in-memory by design; has WAL + snapshots for durability but requires a running server process. Cannot be used SQLite-style.
- **pgvector / pgvectorscale** — requires a running Postgres server
- **Qdrant, Weaviate, Milvus, Neo4j, etc.** — all require server processes

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

### Filter pill behaviour
- **Three-state filter pills** instead of current two-state (on / none):
  - **None** (default) — no filter applied, all DBs shown regardless of this property
  - **On** (green, current behaviour) — only show DBs that HAVE this property
  - **Off** (blacked out / dimmed) — only show DBs that do NOT have this property
- Multiple pills can be active simultaneously; all active filters are ANDed together

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
- Default can be dark (current) or follow system preference via `prefers-color-scheme`
- Light theme should feel clean and professional — white/off-white backgrounds, dark text, same green accent or a slightly deeper green for better contrast on light backgrounds
- All CSS should use variables so the swap is clean; avoid hardcoded hex colors in component styles
| Column | Key | Description |
|---|---|---|
| RAM | `ram` | Runs primarily in RAM — data is held in memory for fast access (e.g. Redis, Memgraph) |
| Persistence | `persistence` | Saves state to disk so data survives restarts — no data loss if server/computer is rebooted |
| Server | `server` | Can be run as a standalone server process, accessible over a network |

### Rename
- **"File-based" → "Embeddable"** — the column already means SQLite-style embedded/in-process use; the label should reflect that. Update column header, tooltip, filter pill label, and any references in the HTML.

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

## File delivered

`db_comparison_table.html` — single self-contained file, open in any browser
