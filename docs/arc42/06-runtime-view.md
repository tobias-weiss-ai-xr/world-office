# 6. Runtime View

## R1 — User opens a document in the editor

```
Browser                docserver (:80)            OCIS / collab (:9300)
   │ GET /word/index.html  │                            │
   │──────────────────────▶│ static_files               │
   │◀── editor JS + wo-bridge                            │
   │ WOPI HTML flow (token) │                            │
   │──────────────────────▶│ wopi.rs ── GetFile ───────▶│
   │◀── editor bootstraps with doc content ◀────────────│
```

Readiness signal in the browser: `#toolbar ..panel.active`.

## R2 — Editing and collaboration

- Edits stay in the browser (WYSIWYG engine); `wo-bridge.js` serializes to
  HTML and pushes via WOPI `PutFile` on save/autosave.
- Collaborative state: `GET /api/documents/<id>/collab/state?session=…` on the
  docserver hub (`collab.rs`). The hub is in-memory; a container restart
  clears it (known operational behavior — "de-poison the hub" runbook step).

## R3 — Format conversion (API)

```
client → POST /api/conversion/convert
         { source_format, target_format, data: base64 }
       ← { status: "Success", data: base64 }   (wo-x2t pipeline)
```
Sync request/response; used by tests, harness roundtrip probes, and internal
tooling. Requires `JWT_SECRET` set on the server.

## R4 — Parity roundtrip probe (test time)

```
harness roundtrip.cjs:
  drive editor via Playwright (apply feature)
  → read editor HTML
  → convert html→docx→html via R3
  → assert marker attributes survived verbatim (13 gates, level3)
```

## R5 — Deployment / health

- Container healthcheck: `curl -f http://localhost:80/health`.
- Liveness probe used by ops/scripts: `GET /healthz`.
- After every editor deploy: pageerror sweep (0 JS page errors expected) and
  `curl`+`cmp` verification of the deployed assets from a known cwd.

## R6 — Agent campaign execution (dev-time)

agentflow orchestrates worker agents in isolated git worktrees against
`config/tasks.json`; workers must make measurable changes (policy: NO CHANGE
IS NOT SUCCESS), run the gate commands, and the orchestrator merges finished
tasks back to main.
