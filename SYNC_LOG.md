# Sync Log

Tracks every merge from `upstream` (`ruma/ruma`) into this fork's `main`
branch: date, upstream commit range, conflicts encountered, and resolution
notes. Append a new entry per sync — do not rewrite history.

Sync cadence: monthly or per notable upstream release (see
[FORK_DIVERGENCE.md](FORK_DIVERGENCE.md)).

## Sync procedure

```sh
git fetch upstream
git merge upstream/main
# resolve any conflicts against documented divergence in FORK_DIVERGENCE.md
cargo build --release
cargo test --workspace
git push origin main
```

Never resolve merge conflicts by hand-splicing both parent versions
line-by-line — use `git merge`'s conflict markers and resolve deliberately,
or `git merge -X ours`/`-X theirs` where appropriate. Hand-splicing has
previously produced syntactically corrupted files that passed `MERGEABLE`
status without conflict markers but broke at build time.

---

## 2026-08-14 — Initial fork

- **Upstream commit:** `2f4413428` (`api: Add support for the GET
  retention/configuration endpoint from MSC1763`), upstream `main` branch.
- **Action:** Forked `ruma/ruma` into `gdnaio/gdna-matrix-sdk` via `gh repo
  fork`. No merge performed — this is the fork point, not a sync.
- **Conflicts:** None (fresh fork).
- **Notes:** Confirmed MIT license unmodified, upstream `CONTRIBUTING.md`
  reviewed (bans LLM-generated upstream contributions — see
  FORK_DIVERGENCE.md). Crate versions at fork point: `ruma-common` 0.19.0,
  `ruma-client-api` 0.24.0, `ruma-federation-api` 0.15.0, `ruma-signatures`
  0.21.0, `ruma-events` 0.34.0.

## 2026-08-14 — Dependency audit and workspace trim (Phases 4-5)

- **Dependency audit:** `cargo tree` + `cargo license` run against full
  upstream workspace (~377 crates). No GPL/AGPL/proprietary licenses found.
  2 crates under MPL-2.0 (`as_variant`, `assert_matches2`) and 1 under
  CDLA-Permissive-2.0 (`webpki-root-certs`, CA cert data) flagged and
  reviewed — no copyleft obligation triggered since these are unmodified
  external dependencies. Full breakdown in `DEPENDENCIES.md`.
- **Workspace trim:** Root `Cargo.toml` `members` changed from glob
  (`crates/*` + `xtask`, 13 crates) to explicit 8-crate list: `ruma-common`,
  `ruma-macros`, `ruma-identifiers-validation`, `ruma-events`,
  `ruma-client-api`, `ruma-federation-api`, `ruma-signatures`,
  `ruma-state-res`. The latter added beyond the original plan at explicit
  user request (needed for room v10+ state resolution in federation).
  `cargo build --release` verified clean on the trimmed workspace.
- **Conflicts:** None — this is a scope/trim change on the fork's own
  workspace config, not an upstream merge.

## 2026-08-14 — First CI run, cargo-sort fix, branch protection (Phase 6 close-out)

- **First CI run:** Fork's GitHub Actions were gated behind GitHub's
  one-time "enable workflows on this fork" confirmation (required manual
  click in the web UI; no API/CLI bypass exists). Once enabled, the first
  `push`-triggered run surfaced a real finding: `Style / External Tools`
  failed because `cargo-sort` had never run against this workspace before —
  root `Cargo.toml`'s `members`/`default-members` lists weren't
  alphabetized, and `ruma-client-api/Cargo.toml` had a stray blank line
  between feature entries (pre-existing upstream artifact). Fixed via
  `cargo sort --workspace --grouped`; both changes are non-semantic
  (confirmed via `cargo build --workspace`).
- **Branch protection:** Applied to `main` requiring all 8 CI jobs
  (Formatting, Clippy, MSRV (1.89), Test / Default Features, Test / All
  Features, Test / Doc Tests, Build (release), Style / External Tools) to
  pass, `strict` mode (branch must be up to date before merge), and
  force-push/branch-deletion disabled.
- Added `workflow_dispatch` trigger to `ci.yml` to allow manual CI runs
  going forward without requiring a fresh push.
