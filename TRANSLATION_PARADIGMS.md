# Translation Paradigms — C++→Rust (World-Office Worker Contract)

> Applies to all agentflow development in this workspace. `wo-*` crates are the
> Rust rewrite of ONLYOFFICE's C++ core (`x2t` parser/model + `sdkjs` editing
> model, see `plan/2026-07-25-onlyoffice-100-percent-equivalence-plan.md` §2).
> The paradigms below are drawn from the software-development-research corpus
> (C++→Rust translation papers, ingested 2026-10-09) and distilled for agents
> translating/porting code. **Structure follows function; these are the rules.**

## The seven paradigms (corpus-grounded)

1. **Sum types over virtual dispatch.** C++ polymorphic pointer trees become
   `enum`s + exhaustive `match`. If a crate models a converter/format/node as
   `Box<dyn Trait>` where the variant set is closed, an enum is the honest type.
   *Why: exhaustive matching is the compiler telling you a converter was added.*
   (arXiv 2609.25121 TRACTOR; 10.1145/3830438.3830958)

2. **Ownership: clone-mutate-store. NEVER borrow across a mutation.**
   The `extract_body` / `store_body` idiom for `DOC_MODEL_STORE` is non-optional.
   C++ `T*`-everywhere becomes owned values + moves. When the borrow checker
   fights you, restructure (pass `&mut` down the call) — do **not** reach for
   `Rc<RefCell<…>>` as a reflex: it is a C++ pointer-the-world habit in disguise.

3. **Fail with `Result`, never panic in library code.** Empty/invalid
   converter input returns `ConversionError`, not `panic!`/`unwrap()`/`expect()`.
   `#![deny(clippy::unwrap_used)]`-style discipline where practical.

4. **Char-index (Unicode) semantics.** Cursor/insert/delete/offset code uses
   `.chars().count()` and CHAR indices, never byte `.len()`. C++ strings are
   byte arrays by default; this is one of the silent semantic gaps.

5. **Verify, don't trust — every translated unit needs tests.**
   *"C-to-Rust Fallacy: Automatic Refactoring != Memory Safety"* (arXiv
   2609.25682): literal translation keeps the C++ architecture and its bugs.
   A converter without roundtrip + error-path tests is un-migrated code. Test
   the translated behavior, not the C++ line-for-line.

6. **`unsafe` is a gate, not a tool.** `unsafe` blocks/seams (FFI, raw ptr,
   deref of a foreign buffer) need: (a) a comment naming the exact invariant
   being upheld, (b) a test that fails if the invariant breaks. Fewer `unsafe`
   sites than the C++ had pointer casts — that is the metric of a translation.

7. **Idiomatic Rust, not transliteration.** Output reads like it was written
   in Rust (iterators, `?`, `impl Default`/`From`, typed ids). Agent workers
   are scored on this: `cargo clippy` clean is the floor, not the ceiling.

## Crash course for agent workers

- C++ `std::unique_ptr<T>` → `Box<T>` · `shared_ptr` → `Rc`/`Arc` (`Weak` for cycles)
- C++ `std::vector` → `Vec`, `std::map` → `BTreeMap`, `std::string` → `String`/`&str`
- C++ exceptions → `Result<T, E>` + `?` (never `panic!` on input)
- C++ `nullptr` → `Option<T>` (`None` handled by `match`/`?`)
- C++ copy-ctor → explicit `.clone()` (no silent copies)
- C++ virtual + inheritance → `enum` (prefer) or `&dyn Trait`
- C++ callbacks/observers → channels (`tokio::sync::watch`/`mpsc`) not `Rc<RefCell<closures>>`
- C++ `co_await` → `.await` with an explicit executor (Tokio)
- C++ destructor/RAII → `Drop` + ownership (but you almost never need `Drop`)

## Reference

- `CXX_TO_RUST_TRANSLATION_GUIDE.md` (research corpus, full guide + examples)
- `plan/2026-07-25-onlyoffice-100-percent-equivalence-plan.md` (§2 x2t/sdkjs C++ heritage)
- `server/AGENTS.md` (build/test/lint, mutation idioms — non-negotiable)
