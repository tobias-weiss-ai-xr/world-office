# 11. Risks and Technical Debts

## Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| **Single-host production** (all containers on 192.168.42.42) | host failure = full outage | backups + documented compose; no HA claim |
| **Stale staging images** (was 2026-10-18 old; caused false "gaps") | wrong converter conclusions | rule: verify against freshly built binary/image, never trust a container's age |
| **Nightly toolchain pin** (AD-8) | CI breakage on nightly regressions | upgrade path known (stable ≥1.92 fixed the ICE) |
| **In-memory collab hub** | restart clears collaboration state | runbook: restart de-poisons; persistence not yet built |
| **Bus factor 1 + agent fleet** | continuity risk | AGENTS.md knowledge bases, this arc42 set, memory systems, gates-as-spec |

## Technical debts (deliberate, with ceilings)

- **Converter ceilings (documented 2026-10-27):** list `start` value not
  emitted; `ilvl` nesting flattened both directions; hand-authored docx
  numbering numIds ≠1/2 degrade to body text in Word.
- **`insert-spreadsheet`** — the last real L0 gate cluster; open task.
- **wo-docserver `lib.rs` is 113 KB** — module split overdue; the crate is
  accreting surfaces (AI, editor docs, collab) that want their own modules.
- **Services are thin shells** — only storage-service has real CRUD; the
  rest await the decomposition trigger (AD-9).
- **Carried follow-ups:** `w:sdt` docx emitter, `textDirection` in wo-ooxml,
  export-to-file, chart 3D depth, optional Home-tab dialog census.
- **Retired debt:** harness roundtrip workarounds (`&nbsp;`→space,
  non-ASCII→numeric refs) — removed 2026-10-27 after the converter fixes
  landed; probes now run verbatim so regressions fail loudly.

## Anti-patterns to avoid (from AGENTS.md)

- Never push to main without `cargo test` on changed crates.
- No new Rust deps without Windows-compile check.
- Never expose WOPI endpoints without auth tokens.
- WASM crates are not `cargo test`-able — use wasm CI path.
