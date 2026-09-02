# Using this fork from `gdna-matrix`

This is the fork-owned usage guide the README's "Usage example" section
links out to for detail. If you're looking for a quick copy-paste
dependency snippet, that section already has it — this document is for
understanding *why* the integration looks the way it does and how to
change it during development.

## What `gdna-matrix` actually imports, and how much

`gdna-matrix` (the homeserver, `/home/crew/repos/matrix-workspace/gdna-matrix`
in this workspace) is a real, active consumer of six of this fork's eight
crates. Import-site counts, from that project's own source tree (not
counting this fork's internal usage of itself):

| Crate | Import sites | Used for |
|---|---|---|
| `ruma_common` | 99 | Core identifiers (`UserId`, `RoomId`, `EventId`, ...), event base types, canonical JSON. The overwhelming majority of this fork's surface area that `gdna-matrix` touches. |
| `ruma_signatures` | 20 | Ed25519 event/request signing and verification — federation PDU signature validation is security-critical, so this crate is load-bearing, not incidental. |
| `ruma_state_res` | 11 | State resolution v2 (room version v10+) — deterministic conflict resolution when two servers disagree about a room's current state. |
| `ruma_events` | 4 | Room/state event type definitions. Fewer call sites than you might expect because `gdna-matrix` mostly works with events through `ruma_common`'s re-exports and its own `src/model.rs` layer (below), not this crate's types directly. |
| `ruma_client_api` | 2 | Client-Server API request/response types. |
| `ruma_federation_api` | 0 | Declared as a workspace dependency; no call sites yet. Federation request/response shapes are currently hand-rolled in `gdna-matrix` rather than using this crate's types — worth revisiting if that duplication grows, not something this fork can decide unilaterally since it's `gdna-matrix`'s own code. |

`gdna-matrix/src/model.rs` **re-exports** this fork's identifier and event
types directly as the homeserver's own model layer, rather than wrapping
them in a local newtype/adapter. The practical consequence: a breaking
change to `ruma-common`'s public API (a renamed field, a changed enum
variant, a different trait bound) is a breaking change to `gdna-matrix`'s
model layer with no shim in between to absorb it. This is worth knowing
before doing an upstream sync that touches `ruma-common` — check
`gdna-matrix/src/model.rs` and its own call sites for fallout, not just
whether this fork's own tests pass.

## How the dependency is actually wired

`gdna-matrix` pins this fork as a **git dependency at a specific commit**,
not a path dependency and not a floating branch:

```toml
# gdna-matrix/Cargo.toml, [workspace.dependencies]
ruma-common = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
```

Why a pinned `rev` rather than `branch = "main"`: an upstream sync landing
on this fork's `main` (see [SYNC_LOG.md](../SYNC_LOG.md)) should never
silently change what `gdna-matrix` builds against. Bumping the pin is a
deliberate, reviewable, one-line diff in `gdna-matrix`'s own `Cargo.toml` —
the same discipline a `Cargo.lock` update already gives you for every
other dependency, applied here explicitly because a git dependency has no
`Cargo.lock`-equivalent safety net of its own without one.

**This fork's own docs do not track which commit `gdna-matrix` currently
pins** — that pin lives in and is owned by `gdna-matrix`'s repository, and
this document does not assert a specific value for it (a stale pinned
commit number here would just be one more thing to keep in sync and get
wrong, the exact problem this documentation pass was written to reduce,
not add to).

## Local co-development: the `[patch]` workflow

Editing this fork and `gdna-matrix` at the same time, without pushing a
commit here for every iteration, needs Cargo's `[patch]` mechanism —
NOT switching the dependency declaration itself to a path dependency (that
would need reverting before every commit, which is easy to forget and ship
by accident). In `gdna-matrix`'s own `Cargo.toml` (not this repository):

```toml
[patch."https://github.com/gdnaio/gdna-matrix-rust-sdk.git"]
ruma-common = { path = "../gdna-matrix-rust-sdk/crates/ruma-common" }
ruma-signatures = { path = "../gdna-matrix-rust-sdk/crates/ruma-signatures" }
# ... one entry per crate you're actively editing; only the ones you list
# here are patched, everything else still resolves from the pinned rev.
```

This assumes the sibling-checkout layout this whole workspace already
uses (`gdna-matrix-rust-sdk` and `gdna-matrix` as sibling directories) —
adjust the relative path if your checkout differs.

## Building this fork's own documentation

```sh
cargo doc          # builds rustdoc for all crates, with an index page
cargo docs-strict   # same, but -Dwarnings -- what CI should eventually run
```

Both are plain `cargo` aliases defined in `.cargo/config.toml`, not
`cargo xtask doc` — upstream's `xtask` package was deliberately dropped
from this fork's workspace during the Phase 5 trim (see
[FORK_DIVERGENCE.md](../FORK_DIVERGENCE.md)), and its doc-building alias
is inlined here directly rather than needing `xtask` restored just to run
one command. `rust-toolchain.toml` pins the nightly these aliases need
(`--cfg docsrs` requires it); you do not need to invoke `rustup run`
yourself, `rustup`'s own toolchain-override file handles that.

Neither alias publishes anywhere — output lands in `target/doc/`,
gitignored, for local review. No CI job currently builds or gates on
rustdoc (`ci.yml` runs fmt, clippy, MSRV, tests, and a release build; doc
warnings are invisible to it). Adding a CI job that runs `cargo docs-strict`
and denies warnings, and deciding where (if anywhere) the output should be
published, is real follow-up work this documentation pass identified but
did not do — see this file's own commit for the honest accounting of what
was and wasn't addressed.
