# Dependencies

Compliance artifact for `gdna-matrix-rust-sdk`'s dependency graph. Generated via
`cargo tree` and `cargo license` against the full (pre-trim) upstream
workspace at fork point (`2f4413428`).

Regenerate after every Phase 5 workspace trim and every upstream sync:

```sh
cargo tree --workspace > dependency-tree.txt   # optional, for local review
cargo license                                   # human-readable summary
cargo license --json                            # machine-readable, for tooling
```

## Summary

- **Total resolved crates:** ~377 (pre-trim, full upstream workspace including
  `ruma-client`, `uniffi` FFI bindings, `xtask`, and dev/test-only deps).
  This count dropped after Phase 5 trimmed the workspace to its current
  8 members (`ruma-common`, `ruma-macros`, `ruma-identifiers-validation`,
  `ruma-events`, `ruma-client-api`, `ruma-federation-api`,
  `ruma-signatures`, `ruma-state-res` — see `FORK_DIVERGENCE.md`'s Kept
  table for why each one stayed), but this document has not been
  regenerated against the trimmed workspace since — the figures below are
  still the pre-trim, full-upstream audit. Re-running `cargo tree`/
  `cargo license` against the current 8-crate workspace and updating this
  file is tracked as separate follow-up work, not done as part of this
  documentation pass (this pass fixed what the document CLAIMED, not what
  it never re-measured).
- **GPL/AGPL/proprietary findings:** None.
- **Copyleft findings requiring attention:** 2 crates under MPL-2.0
  (`as_variant`, `assert_matches2`) — see below. File-level copyleft only
  triggers on modification of the MPL-covered crate's own source; we consume
  these unmodified as external dependencies, so no obligation is triggered.
- **Non-standard but permissive:** `webpki-root-certs` (CDLA-Permissive-2.0)
  — a CA root certificate data bundle, not code; permissive data license.

## License breakdown (by crate count)

| License expression | Crate count |
|---|---|
| Apache-2.0 OR MIT | 237 |
| MIT | 60 |
| Unicode-3.0 | 18 |
| Apache-2.0 OR Apache-2.0 WITH LLVM-exception OR MIT | 16 |
| Apache-2.0 OR MIT OR Zlib | 6 |
| MIT OR Unlicense | 6 |
| Zlib | 4 |
| Apache-2.0 | 3 |
| Apache-2.0 OR ISC OR MIT | 3 |
| BSD-3-Clause | 3 |
| Apache-2.0 OR BSD-2-Clause OR MIT | 2 |
| Apache-2.0 OR LGPL-2.1-or-later OR MIT | 2 |
| ISC | 2 |
| MPL-2.0 | 2 |
| MPL-2.0 (see below) | — |
| Apache-2.0 AND ISC | 1 |
| Apache-2.0 OR BSD-1-Clause OR MIT | 1 |
| Apache-2.0 OR BSL-1.0 | 1 |
| Apache-2.0 OR CC0-1.0 OR MIT-0 | 1 |
| CDLA-Permissive-2.0 | 1 |
| 0BSD OR Apache-2.0 OR MIT | 1 |
| Compound expressions (aws-lc-sys, aws-lc-rs, encoding_rs, unicode-ident) | 4 |
| UNKNOWN (workspace-internal, unpublished) | 1 (`xtask`) |

Note: `Apache-2.0 OR LGPL-2.1-or-later OR MIT` (2 crates) is a license
*choice* — we consume these under the Apache-2.0/MIT terms, not LGPL. No
LGPL obligation applies since we're not exercising that license option.

## Flagged crates (manual review)

| Crate | License | Notes |
|---|---|---|
| `as_variant` | MPL-2.0 | Small `matches!`-style helper macro. Unmodified external dep — no copyleft obligation. |
| `assert_matches2` | MPL-2.0 | Test/dev-only assertion macro. Unmodified external dep — no copyleft obligation. Likely dropped or dev-only after Phase 5 trim. |
| `webpki-root-certs` | CDLA-Permissive-2.0 | CA root cert bundle (data, not code), pulled in transitively via TLS stack. Permissive. |

## No GPL/AGPL/proprietary code found

Confirmed via `cargo license --json` full scan: zero crates carry GPL-2.0,
GPL-3.0, AGPL-3.0, or any proprietary/non-OSI license. No vendored
Element-specific or other proprietary code identified in the dependency
tree (only Ruma's own crates and their public, permissively-licensed
upstream dependencies).

## Re-audit trigger

Re-run this audit whenever:
- `Cargo.lock` changes materially (new dependency, major version bump)
- Before each upstream sync merge lands on `main`
- Before any crates.io publish decision is revisited (currently: no publish,
  path/git dependency only — see FORK_DIVERGENCE.md)
