# 5. Building Block View

## Level 1 — the five subsystems

```
World-Office
├── 1. Rust core (26 crates)     format parse/render/convert, protocols
├── 2. wo-docserver              THE deployment unit: editor UI host + WOPI +
│                                conversion API + collab hub (axum binary)
├── 3. Editor frontends          browser apps (word WYSIWYG live; React apps)
├── 4. Services                  thin axum/tokio microservices (mostly future
│                                decomposition surface)
└── 5. Integrations & desktop    OpenCloud, Nextcloud, Android, Tauri, WASM
```

## Level 2 — Rust core crate groups (`server/core/crates/`)

| Group | Crates | Role |
|-------|--------|------|
| Shared base | wo-common, wo-unicode, wo-office-utils | errors, test harness, encodings |
| Text formats | wo-txt, wo-rtf, wo-html, wo-fb2, wo-epub | `FormatRoundtrip` parsers |
| Office formats | wo-ooxml (DOCX/XLSX/PPTX), wo-odf, wo-msbinary, wo-hwp | the fidelity-critical family |
| Fixed-layout | wo-pdf, wo-pdf-render, wo-djvu, wo-xps, wo-ofd, wo-visio | read/render |
| Conversion | **wo-x2t** (+ wo-x2t-wasm) | 27 native converters, chaining |
| Rendering | wo-renderer (+ wasm), wo-fonts, wo-raster, wo-docx-renderer | canvas pipeline |
| Editing models | wo-sheet, wo-slide, wo-chart, wo-formula, wo-spell | per-domain models |
| Protocols/apps | **wo-docserver**, wo-wopi, wo-webdav, wo-route, wo-ooxml-ops | network + operations layer |

## Level 2 — wo-docserver modules (`core/crates/wo-docserver/src/`)

| Module | Responsibility |
|--------|----------------|
| `lib.rs` (113 KB) | app assembly, routes, conversion endpoints |
| `wopi.rs` | WOPI protocol implementation (largest surface) |
| `collab.rs` | collaboration hub state endpoints |
| `api_ext.rs`, `editor_docs.rs` | extended APIs, editor document plumbing |
| `store.rs` | local document store (`DOCSERVER_DATA_DIR`) |
| `static_files.rs` | editor UI hosting from `EDITOR_UI_DIR` |
| `docx_util.rs`, `ai.rs`, `config.rs` | docx helpers, AI surface, env config |

## Level 2 — frontend (`apps/web/apps/`, `packages/`)

- `documenteditor-wysiwyg/` — **the live word editor** (vendored vanilla JS:
  `index.html editor.js i18n.js style.css`, deployed as those 4 files), with
  `wo-bridge.js` as the WOPI adapter.
- `*-react/` editors (spreadsheet, presentation, pdf, visio, documenteditor),
  `editor-shell/`, `api/`, `common/`.
- `packages/` (@world-office): editor-common, editor-stores, design-system,
  collaboration-client/-react, sdk-bridge, plugin-sdk, i18n.

## Level 3 — the converter fidelity path (parity frontier)

```
editor HTML ──wo-html parse──▶ HTML model (Block/Inline, align, entities)
   │                                       │
   ▼ wo-x2t html_to_ooxml                  ▼ wo-x2t docx_body_to_html_blocks
OOXML model (DocxRun props, numPr,          │
hyperlink_rid, TextAlignment) ◀─wo-ooxml────┘
   │            parse (document.xml, numbering.xml,
   │            document.xml.rels incl. hyperlinks)
   ▼ wo-ooxml serialize (docx zip: document.xml, numbering.xml,
     styles.xml, rels with TargetMode="External", media)
```

Key invariant: **all fidelity fixes stay in the converters + format models**
(wo-html, wo-ooxml, wo-x2t) — never in the editor or the docserver.

## Services (`server/services/`)

Thin binaries wrapping library crates (axum + tokio): api-gateway,
identity-service (JWT/OAuth2), session-service, storage-service (SQLite
metadata + disk blobs — the only one with full CRUD + tests),
conversion-service (calls wo-x2t), coauthoring-service, mcp-server,
admin-panel (TS dashboard). These are a **future decomposition surface**, not
the current production path — production traffic goes through wo-docserver.
