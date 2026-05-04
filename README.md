# Vector & graph database comparison

This repository holds a **standalone comparison table** for twenty-five vector, graph, and multi-model databases. Open `db_comparison_table.html` in a browser to explore sortable columns, feature filters, search, column visibility, light/dark themes, CSV export, and caveat tooltips where a “yes” needs nuance.

There is **no build step** and no server required. The page loads data from `db_comparison_data.js` next to it (same folder).

## Quick start

1. Clone or download this folder.
2. Open `db_comparison_table.html` in a modern desktop browser (double-click or “Open with…”).

For local file access, some browsers restrict `file://` scripts; if the table stays empty, serve the directory briefly, for example:

```bash
python3 -m http.server 8765
```

Then visit `http://localhost:8765/db_comparison_table.html`.

## What you get

- **Sortable columns** — click headers to toggle ascending/descending order.
- **Feature pills** — three-state filters (no filter / require “on” / exclude “off”) for boolean-style capabilities.
- **Search** — narrows rows by name, tagline, release date, languages, or OS support text.
- **Caveats** — some “yes” cells show an orange indicator; hover for short explanations (license changes, persistence defaults, etc.). Text lives in the `notes` field in the data file, not in a separate document.
- **Themes** — cycles system → light → dark via the header control. **Theme choice is not saved**; reload always starts from system until you click again.
- **Columns** — show or hide column groups. **Search, filters, and column visibility** persist in the browser (`localStorage`, key `dbcmp`).
- **Reset all** — clears search, every feature filter, and column visibility back to the default set.
- **CSV** — `↓ CSV` downloads the **full** embedded comparison as `db_comparison.csv` (same bytes as `window.DB_CSV_DATA`). It is **not** a snapshot of the filtered/sorted on-screen rows.

Fonts (Space Mono, DM Sans) load from Google Fonts; otherwise the experience is self-contained.

## Repository layout

| File | Role |
|------|------|
| `db_comparison_table.html` | UI: table, filters, tooltips, persistence, export. |
| `db_comparison_data.js` | Data: CSV string `window.DB_CSV_DATA` parsed on load. **Edit this file** to add or change databases and fields. |
| `context.md` | Dense notes for **LLM assistants** (CSV keys, boolean-field semantics, UI behavior, `localStorage` shape, inclusion policy, known gaps). |
| `README.md` | This file — overview and usage for people reading the repo. |

## Updating the data

Edit `db_comparison_data.js` only. The embedded CSV uses a fixed header row; boolean columns use `1` / `0`; fields with commas are double-quoted; the `notes` column holds optional JSON (CSV-escaped) for per-column caveat text.

Exact column keys, formatting rules, and **what each boolean column is supposed to mean** are in `context.md` (**CSV schema** and **Boolean keys**).

## Scope and limitations

The set of products and the boolean judgments are **opinionated heuristics** for comparison, not vendor specifications. Always confirm licensing, deployment, and features on each product’s official site.

The table lists **25** curated systems (vector, graph, and multi-model). **Kuzu** is intentionally omitted; **Vespa** and **SurrealDB** were added to the original list. Other engines (for example ClickHouse or Turbopuffer) were left out when they did not fit the same “retrieval + graph” comparison frame.

The table is aimed at **desktop** use: expect horizontal scrolling on small screens. Tooltip behavior near screen edges may still need polish.
