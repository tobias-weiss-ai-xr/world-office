# 10. Quality Requirements

Quality tree (priority-ordered) with the concrete, currently-enforced checks:

```
Q1 Functional parity (OnlyOffice reference)
 ├─ Q1.1 Feature coverage: ledger L0 unknowns ≈ 0 (242 features scored;
 │       last real L0 gap cluster: insert-spreadsheet)
 ├─ Q1.2 Dialog/menu parity: OO dialog census complete; chrome parity tasks
 │       done (CHROME1–5), zero pageerrors in prod sweeps
 └─ Q1.3 Regression protection: static JS gate pins (substring, no regex)
Q2 Format fidelity (converter)
 ├─ Q2.1 docx roundtrip: 13/13 probes survive verbatim (no harness
 │       workarounds — retired 2026-10-27)
 ├─ Q2.2 Unit gates: wo-html 112 / wo-ooxml 66 / wo-x2t 438 green
 └─ Q2.3 Ceiling management: known, documented losses only (list start
 │       value, ilvl depth, foreign numbering numIds)
Q3 Security
 ├─ Q3.1 WOPI/JWT: no unauthenticated WOPI endpoints; JWT_SECRET shared
 ├─ Q3.2 Weekly audit job (cargo audit + pnpm audit --audit-level=high)
 └─ Q3.3 License hygiene: AGPL core vs. private enterprise repo split
Q4 Operability
 ├─ Q4.1 Health endpoints (/health container check, /healthz probe)
 ├─ Q4.2 Deploy verification: curl+cmp asset check + pageerror sweep
 └─ Q4.3 Observability stack available (Grafana/Prometheus/Loki/Tempo
         under server/observability)
Q5 Maintainability
 ├─ Q5.1 Crate boundaries (26 core crates, FormatRoundtrip contracts)
 ├─ Q5.2 fmt + clippy + Biome clean; tests required for all changes
 └─ Q5.3 Agent-verifiable tasks (machine-checkable acceptance)
Q6 Portability
 ├─ Q6.1 Windows dev via WSL documented
 └─ Q6.2 WASM targets build in dedicated CI workflow
```

Not yet formally measured: latency/throughput budgets (baseline notes in
`plan/performance.md`), availability targets (single-host today).
