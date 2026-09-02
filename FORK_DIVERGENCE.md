# Fork Divergence

This document tracks how `gdna-matrix-rust-sdk` differs from upstream
[`ruma/ruma`](https://github.com/ruma/ruma), and why. Update this file
whenever a change is made that is not a straight upstream merge.

## Origin

- Forked from `ruma/ruma` at commit `2f4413428` (upstream `main`, 2026-08-14).
- License: MIT, retained unmodified. See [LICENSE](LICENSE).
- Fork created via `gh repo fork ruma/ruma --org gdnaio --fork-name gdna-matrix-sdk`,
  then renamed to `gdna-matrix-rust-sdk` on 2026-08-27 (commit `9afae7b36`,
  see [SYNC_LOG.md](SYNC_LOG.md)). The rename commit changed only this
  repository's name and one line of `README.md`; no reason was recorded at
  the time, and none is asserted here — a plausible guess (disambiguating
  from a sibling `gdna-matrix-js-sdk` fork in the same workspace) is not
  the same as a documented one, and this file only states what the commit
  history actually shows.

## Purpose

Maintained independently to support GDNA's Matrix homeserver implementation.
The goal is to consume Ruma's protocol types (events, Client-Server API,
Server-Server/federation API, signatures) as a stable dependency layer,
trimming the workspace to only the crates the homeserver actually needs
rather than tracking the full upstream workspace.

## Upstream contribution policy — read before submitting fixes upstream

Upstream `ruma/ruma`'s `CONTRIBUTING.md` **explicitly prohibits LLM-generated
contributions** (code, docs, or issues) in PRs submitted to the upstream
project. This does **not** affect this fork itself — MIT permits full reuse,
modification, and redistribution regardless of how changes were produced.
It **does** block the "contribute homeserver-agnostic fixes back to
`ruma/ruma`" item under Ongoing Maintenance for any AI-assisted change:
such fixes must be authored/reviewed by a human without AI assistance before
they can be submitted as an upstream PR. Track any fixes we'd like to
contribute back in a separate list and flag human-authorship status before
opening a PR against `ruma/ruma`.

## Workspace changes from upstream

Root `Cargo.toml` `members` changed from the glob `["crates/*", "xtask"]`
(all 13 crates + build tooling) to an explicit 8-crate list. `xtask` (CI/dev
tooling, not a library) is dropped entirely — it isn't needed to build or
consume the kept crates and pulls in extra build-time-only tooling deps.

## Kept / excluded crates and rationale

**Kept (8):**

| Crate | Why |
|---|---|
| `ruma-common` | Core Matrix types (identifiers, events base types, serialization). Required by everything. |
| `ruma-macros` | Proc-macro crate. **Hard build dependency of `ruma-common`** — not optional, the plan's original crate list omitted this transitive requirement. |
| `ruma-identifiers-validation` | ID validation logic. **Hard build dependency of both `ruma-common` and `ruma-macros`.** |
| `ruma-events` | Room/state event type definitions. **Hard dependency of `ruma-client-api`, `ruma-federation-api`, `ruma-signatures`, and `ruma-state-res`** — the original plan didn't list it, but nothing above it builds without it. |
| `ruma-client-api` | Client-Server API request/response types. Explicitly requested in the plan. |
| `ruma-federation-api` | Server-Server (federation) API request/response types. Explicitly requested in the plan. |
| `ruma-signatures` | Ed25519 signing/verification for event and request signing — required for federation PDU signature validation (security-critical per project policy). Explicitly requested in the plan ("if compatible" — confirmed compatible). |
| `ruma-state-res` | State resolution v2 algorithm (room version v10+). **Added beyond the original plan's crate list** at explicit user request during Phase 5 — a homeserver handling federation on room v10+ needs this for deterministic conflict resolution of room state. No dependency conflicts with the other 7; verified via `cargo build --release`. |

**Excluded (4):**

| Crate | Why excluded |
|---|---|
| `ruma` | Umbrella/re-export crate pulling in every other crate (including all excluded ones below) as optional features. Not needed — we depend on the specific crates directly. |
| `ruma-html` | Optional HTML sanitization helper for `m.room.message` formatted bodies. Not a direct workspace member, but pulled in transitively as a path dependency of `ruma-events`'s optional `html` feature — `cargo metadata` resolves it as a de facto 9th package despite this table listing it as excluded. Feature-gated in `ruma-events` itself, so it costs nothing unless that feature is enabled. |
| `ruma-appservice-api` | Application Service API types. Deferred per plan — no AS support planned initially. |
| `ruma-push-gateway-api` | Push notification gateway types. Not needed for core homeserver federation/C-S functionality. |
| `ruma-identity-service-api` | Identity Service (3PID lookup) API types. Not needed initially; add later if 3PID/identity-server integration is required. |

`ruma-state-res` was excluded in the original plan and added back into the
**Kept** table above during Phase 5 at explicit user request — it is not
listed here a second time; an earlier version of this document listed it
in both tables (struck through in this one), which was a genuine
inconsistency in the document rather than a real dual state of the crate.

Note: no separate `ruma-client` crate exists in this workspace — upstream's
full client implementation lives in a separate repository
(`ruma/ruma-client`), not in `ruma/ruma`'s workspace, so there was nothing
to exclude here despite the plan mentioning it.

`workspace.dependencies` entries for `ruma-appservice-api` and `ruma-html`
remain declared (unused, harmless) since Cargo does not error on unused
workspace-dependency declarations; removing them is cosmetic cleanup, not
a build requirement.

## Custom code changes

- **2026-09-02 — Documentation correction and doc-build tooling fix.**
  `cargo xtask doc` (the documented way to build this fork's rustdoc)
  broke silently after the Phase 5 workspace trim dropped `xtask` from
  `members` — the `.cargo/config.toml` alias survived pointing at a
  package that no longer existed. Replaced with plain `cargo doc`/
  `cargo docs-strict` aliases that reproduce the same
  `--enable-index-page --cfg docsrs` behaviour without needing `xtask`
  restored, matching the trim's own intent rather than reversing it.
  Corrected three stale/false claims discovered by direct
  investigation: `README.md`'s usage example named a nonexistent sibling
  directory (`../gdna-matrix-sdk/...`, pre-rename) and omitted that
  `gdna-matrix` actually consumes this fork via a pinned **git**
  dependency, not a path dependency; `README.md`'s consumer-integration
  status claimed `gdna-matrix` was "not yet a Cargo project" when it is a
  real, 119-import-site consumer; and this file's own Kept/Excluded
  tables listed `ruma-state-res` in both, a genuine internal
  contradiction rather than a real dual state. `DEPENDENCIES.md` still
  reports the pre-trim, full-upstream audit numbers and says so
  explicitly now, rather than the previous text's false claim that the
  trim would leave only 4 crates (it kept 8) — re-running the audit
  against the trimmed workspace is separate, not-yet-done follow-up
  work, named here rather than silently assumed complete.

- **2026-08-14 — Canonical JSON spec-vector regression tests** (`crates/ruma-common/src/canonical_json.rs`): Added `spec_examples_canonical_json` pinning 7 of the 9 published Matrix spec canonical-JSON examples (empty object, key sort, Unicode key sort/content, `\u` escape decoding, `null`) that weren't previously covered verbatim (one, the `auth`/`mxid` example, was already covered by the pre-existing `check_canonical_sorts_keys` test).

  **Known, intentional divergence from the spec's literal worked example:** the spec's `{"a": -0, "b": 1e10}` → `{"a":0,"b":10000000000}` example assumes a JSON parser that classifies whole-valued floats as integers during parsing (true of Python's `json` module, the spec's reference implementation). `serde_json::Value` does not do this — both `-0` and `1e10` parse as `f64` regardless of whether the value is a whole number. Ruma's `TryFrom<JsonValue> for CanonicalJsonValue` correctly rejects true floats per the spec's own stated intent ("Float values are not permitted by this encoding"), so this specific input produces `CanonicalJsonError::InvalidType("float")` rather than silently coercing to an integer. This is documented as a deliberate choice, not a bug: refusing ambiguous numeric input is the safer behavior on a security-critical signing path. Test: `spec_example_numeric_normalization_diverges_by_design`, same file. Callers must construct genuine integers (`json!(0)`, not `json!(-0)` or `json!(1e10)`) before values reach canonical-JSON serialization.
  - Ed25519 signing/verification against the spec's official JSON Signing and Event Signing test vectors (signing key seed, expected signatures) is already covered by pre-existing `ruma-signatures` doc-tests (`sign_json`, `sign_event`, `hash_and_sign_event`, `verify_event`, `verify_json`) — confirmed passing, no new tests needed for those vectors.

## Sync cadence

Monthly, or on notable upstream releases (security fixes, protocol/MSC
updates). See [SYNC_LOG.md](SYNC_LOG.md) for merge history.
