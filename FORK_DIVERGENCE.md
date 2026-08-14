# Fork Divergence

This document tracks how `gdna-matrix-sdk` differs from upstream
[`ruma/ruma`](https://github.com/ruma/ruma), and why. Update this file
whenever a change is made that is not a straight upstream merge.

## Origin

- Forked from `ruma/ruma` at commit `2f4413428` (upstream `main`, 2026-08-14).
- License: MIT, retained unmodified. See [LICENSE](LICENSE).
- Fork created via `gh repo fork ruma/ruma --org gdnaio --fork-name gdna-matrix-sdk`.

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

**Excluded (5):**

| Crate | Why excluded |
|---|---|
| `ruma` | Umbrella/re-export crate pulling in every other crate (including all excluded ones below) as optional features. Not needed — we depend on the specific crates directly. |
| `ruma-html` | Optional HTML sanitization helper for `m.room.message` formatted bodies. Feature-gated, not required for core protocol types. Can be added later if rich-text rendering is needed. |
| `ruma-appservice-api` | Application Service API types. Deferred per plan — no AS support planned initially. |
| `ruma-state-res` | ~~Excluded~~ — see Kept table above; added back in during Phase 5. |
| `ruma-push-gateway-api` | Push notification gateway types. Not needed for core homeserver federation/C-S functionality. |
| `ruma-identity-service-api` | Identity Service (3PID lookup) API types. Not needed initially; add later if 3PID/identity-server integration is required. |

Note: no separate `ruma-client` crate exists in this workspace — upstream's
full client implementation lives in a separate repository
(`ruma/ruma-client`), not in `ruma/ruma`'s workspace, so there was nothing
to exclude here despite the plan mentioning it.

`workspace.dependencies` entries for `ruma-appservice-api` and `ruma-html`
remain declared (unused, harmless) since Cargo does not error on unused
workspace-dependency declarations; removing them is cosmetic cleanup, not
a build requirement.

## Custom code changes

_(None yet — fork is currently an unmodified mirror of upstream `main` at the
commit above. Log any divergent code changes here as they land, with
rationale and the commit(s) involved.)_

## Sync cadence

Monthly, or on notable upstream releases (security fixes, protocol/MSC
updates). See [SYNC_LOG.md](SYNC_LOG.md) for merge history.
