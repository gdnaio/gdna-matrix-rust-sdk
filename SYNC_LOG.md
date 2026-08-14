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
