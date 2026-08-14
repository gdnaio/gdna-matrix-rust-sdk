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

_(To be filled in during Phase 5 — Workspace Cleanup.)_

Planned: trim root `Cargo.toml` `members` to `ruma-common`, `ruma-client-api`,
`ruma-federation-api` (+ `ruma-signatures` if compatible). Candidates for
removal: `ruma-client` (full client implementation, not needed for a
homeserver), `ruma-appservice-api` (deferred until application-service
support is needed).

## Kept / excluded crates and rationale

_(To be filled in during Phase 5.)_

## Custom code changes

_(None yet — fork is currently an unmodified mirror of upstream `main` at the
commit above. Log any divergent code changes here as they land, with
rationale and the commit(s) involved.)_

## Sync cadence

Monthly, or on notable upstream releases (security fixes, protocol/MSC
updates). See [SYNC_LOG.md](SYNC_LOG.md) for merge history.
