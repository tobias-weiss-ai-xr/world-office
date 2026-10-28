# 12. Glossary

| Term | Meaning |
|------|---------|
| **arc42** | Template for architecture documentation (this set, §1–§12). |
| **census** | Machine-collected inventory of the reference suite's features/controls/dialogs (`wo-test-harness/census/`); ground-truth JSONs are read-only. |
| **collab hub** | In-memory collaboration state inside wo-docserver (`collab.rs`), exposed via `/api/documents/<id>/collab/state`. |
| **EDITOR_UI_DIR** | Directory the docserver serves editor apps from (`/app/editor-ui`); volumes override the baked-in assets. |
| **FormatRoundtrip** | Trait contract implemented by the 16 format parser crates: parse to model, serialize back, losslessly where possible. |
| **LEDGER / L0–L4** | Parity ledger: 242 features in 16 clusters scored L0 (unknown) → L4 (verified persistent). L0 ≈ 0 is the completeness contract. |
| **OCIS** | ownCloud Infinite Scale — the OpenCloud file/identity backend; WOPI host. |
| **OnlyOffice (OO)** | The reference suite the parity program measures against. |
| **pageerror sweep** | Post-deploy browser check: load the editor, require zero JS page errors. |
| **parity deficit** | Count of census features with no World-Office counterpart (152 → 3 → structurally zero over the 2026-10 campaign arc). |
| **roundtrip gate** | Probe that applies a feature in the real editor, converts html→docx→html through the real endpoint, and asserts markers survived. |
| **wo-x2t** | Conversion orchestrator crate: 27 native converters, chain support; the html↔docx path is the parity frontier. |
| **WOPI** | Web Application Open Platform Interface — the HTTP protocol between editor (host) and storage (OCIS). |
| **wo-bridge.js** | WOPI adapter gluing the vendored WYSIWYG editor to the docserver. |
| **agentflow (af)** | Campaign orchestrator that dispatches implementation tasks to worker agents in isolated git worktrees (`config/tasks.json`). |
| **wo-docserver** | The canonical Rust document server container: editor UI host, WOPI endpoints, conversion API, collab hub. |
