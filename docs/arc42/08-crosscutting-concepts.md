# 8. Cross-cutting Concepts

## Document models and the FormatRoundtrip trait

Every format crate implements parse → shared model → serialize. Fidelity work
is contract-first: extend the model (e.g. `Paragraph.align`,
`DocxBody.hyperlink_rels`), wire parse + serialize, then pin with roundtrip
gate tests. Conventions that have emerged:

- **Entity policy (wo-html):** XML's five named entities stay named; known
  HTML named entities become numeric refs; unknown names degrade to literal
  text. A parse error is never acceptable.
- **numPr convention (wo-ooxml):** numId 1 = bullet, numId 2 = decimal,
  style `ListParagraph`; `word/numbering.xml` is emitted only when some
  paragraph carries numPr (documents without lists keep their old byte shape).
- **Relationships:** hyperlink rels ride the same rels vector as images
  (`rIdLnk{n}` ids), with `TargetMode="External"`.

## Fidelity gates (testing concept)

Three layers, all machine-checkable:

1. **Unit gates** — `cargo test` per crate (wo-html 112, wo-ooxml 66,
   wo-x2t 438 + converter gap suites like `tests/converter_gaps.rs`).
2. **Roundtrip probes** — `wo-test-harness` roundtrip.cjs drives the real
   editor via Playwright, converts through the real HTTP endpoint, asserts
   markers survive (13/13, no sanitization allowed).
3. **Static JS gates** — substring pins (`src.includes(...)`, no regex) on
   the landed editor bundle, because regex escapes in JS strings are a
   recurring trap.

## Parity methodology

Feature census of OnlyOffice (ground truth JSONs, read-only) → ledger
(242 features, 16 clusters, L0 unknown … L4 verified) → gap campaigns
(agentflow) → gates land with each fix → harness reruns re-score. Noise
filtering rules (regex, backstage exclusion lists, statusbar raw ids) live in
`wo-test-harness/census/ledger/build.py`.

## Editor conventions

- Ribbon overflow scrolls by **translation**; rb-menu clicks in Playwright
  must be synthetic (`page.evaluate(el.click())`).
- Dialogs live behind split-button menu items; dialog frame lookup is
  ONE-SHOT (`findDialogFrame()`).
- i18n keys must exist in BOTH `en` and `de`.
- Landed JS must pass `node --check`.
- TDZ rule: guard code goes after the helper's `const` declarations.

## Operations conventions

- Prod pageerror sweep after every deploy; `NO_PROXY=127.0.0.1,localhost` for
  local Node fetches; zombie probe ports hunted with `ss -tlnp`.
- Deploys verified with `curl` + `cmp` **from a known cwd** (relative paths
  have bitten before).
- `docker restart opencloud-compose-docserver-1` de-poisons the collab hub.
