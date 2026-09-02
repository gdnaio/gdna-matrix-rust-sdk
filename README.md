# gdna-matrix-rust-sdk

> **This is a fork of [Ruma](https://github.com/ruma/ruma), copyright the Ruma
> contributors, licensed under [MIT](LICENSE).** It is maintained independently
> to support GDNA's Matrix homeserver implementation. See
> [FORK_DIVERGENCE.md](FORK_DIVERGENCE.md) for what has changed from upstream
> and why, and [SYNC_LOG.md](SYNC_LOG.md) for the upstream merge history.
>
> Original project README follows.

---

# Ruma – Your home in Matrix

A set of [Rust] crates (libraries) for interacting with the [Matrix] chat
network.

[website] • [chat] • [documentation][docs] ([unstable][unstable-docs])

[Rust]: https://rust-lang.org/
[Matrix]: https://matrix.org/
[website]: https://ruma.dev/
[chat]: https://matrix.to/#/#ruma:matrix.org
[docs]: https://docs.rs/ruma/
[unstable-docs]: https://docs.ruma.dev/ruma/

## Getting started

If you want to build a Matrix client or bot, have a look at [matrix-rust-sdk].
It builds on Ruma and includes handling of state storage, end-to-end encryption
and many other useful things.

For homeservers, bridges and harder-to-categorize software that works with
Matrix, you're at the right place. To get started, add `ruma` to your
dependencies:

```toml
# crates.io release
ruma = { version = "0.15.0", features = ["..."] }
# git dependency
ruma = { git = "https://github.com/ruma/ruma", branch = "main", features = ["..."] }
```

`ruma` re-exports all of the other crates, so you don't have to worry about
them as a user. Check out the documentation [on docs.rs][docs] (or on
[docs.ruma.dev][unstable-docs] if you use use the git dependency).

You can find a low level Matrix client in the [ruma-client repository](https://github.com/ruma/ruma-client).

You can also find a small number of examples in our dedicated
[ruma-examples repository](https://github.com/ruma/ruma-examples).

[matrix-rust-sdk]: https://github.com/matrix-org/matrix-rust-sdk#readme

## Status

Ruma 0.15.0 supports all events and REST endpoints of Matrix 1.18.

Only room versions enforcing canonical JSON (introduced with room version 6) are
supported. Room versions 1 through 5 are supported on a best effort basis, but a
missing feature or an incompatibility with a homeserver implementation are not
considered bugs. Clients should be able to work with those room versions,
granted parts of the room might break in some unconventional cases, but
homeservers based on Ruma **should not** advertise support for them.

Various changes from in-progress or finished MSCs are also implemented, gated
behind the `unstable-mscXXXX` (where `XXXX` is the MSC number) Cargo features.

## Usage example

See [`docs/USAGE.md`](docs/USAGE.md) for the full picture — which crates
`gdna-matrix` actually imports and why, the pinned-`rev`-vs-branch
rationale, and the `[patch]` workflow for local co-development. The short
version:

`gdna-matrix` (the homeserver) does not consume this via a path dependency —
it pins these crates as a **git dependency** at a specific commit, in its own
`Cargo.toml`:

```toml
[workspace.dependencies]
ruma-common = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
ruma-client-api = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
ruma-federation-api = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
ruma-signatures = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
ruma-events = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
ruma-state-res = { git = "https://github.com/gdnaio/gdna-matrix-rust-sdk.git", rev = "<commit>" }
```

A pinned `rev`, not a floating `branch`, so an upstream sync into this fork's
`main` never silently changes what a consumer builds against — bumping the
pin is a deliberate, reviewable step in the consuming project, the same way a
`Cargo.lock` update is. See that project's own `README.md` for why the pin
is a commit rather than a branch, and its `[patch]` recipe for pointing at a
local checkout of this fork during co-development (edit a crate here,
`[patch."https://github.com/gdnaio/gdna-matrix-rust-sdk.git"]` it to a local
path in the consumer, iterate without pushing every change first).

If you are consuming this fork from a **sibling checkout** on the same
machine rather than pinning a git rev (e.g. for that same local
co-development workflow, or a from-scratch integration), a path dependency
works too — point it at this repository's actual current directory name:

```toml
[dependencies]
ruma-common = { path = "../gdna-matrix-rust-sdk/crates/ruma-common" }
ruma-client-api = { path = "../gdna-matrix-rust-sdk/crates/ruma-client-api" }
ruma-federation-api = { path = "../gdna-matrix-rust-sdk/crates/ruma-federation-api" }
ruma-signatures = { path = "../gdna-matrix-rust-sdk/crates/ruma-signatures" }
ruma-state-res = { path = "../gdna-matrix-rust-sdk/crates/ruma-state-res" }
```

Minimal round-trip example — canonical JSON serialize/deserialize (the form
used for event signing):

```rust
use ruma_common::{CanonicalJsonObject, CanonicalJsonValue};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let event = serde_json::json!({
        "room_id": "!x:example.org",
        "sender": "@alice:example.org",
        "type": "m.room.message",
        "content": { "body": "hello" },
    });

    // Convert to CanonicalJsonValue: sorted keys, no whitespace, UTF-8.
    let canonical: CanonicalJsonObject = match CanonicalJsonValue::try_from(event)? {
        CanonicalJsonValue::Object(obj) => obj,
        _ => unreachable!(),
    };

    let serialized = serde_json::to_string(&canonical)?;
    println!("{serialized}");
    // {"content":{"body":"hello"},"room_id":"!x:example.org","sender":"@alice:example.org","type":"m.room.message"}

    // Round-trip back.
    let deserialized: CanonicalJsonObject = serde_json::from_str(&serialized)?;
    assert_eq!(canonical, deserialized);

    Ok(())
}
```

For Ed25519 event signing, see [`ruma_signatures::hash_and_sign_event`
docs](https://docs.rs/ruma-signatures) — this fork's `FORK_DIVERGENCE.md`
and `crates/ruma-common/src/canonical_json.rs` test module document the
canonical-JSON spec-vector validation this fork performs.

## Consumer integration status

- **`gdna-matrix`** (homeserver, Rust): a real Cargo workspace, and a real
  consumer of this fork — pinned as a git dependency (see the Usage example
  above) at `rev = 34e33012f...`, the commit immediately before this repo
  was renamed from `gdna-matrix-sdk` to `gdna-matrix-rust-sdk`. That pin
  still resolves today only via GitHub's automatic rename redirect, not
  because it names the current repository directly — updating it is tracked
  as separate, deliberate work in `gdna-matrix`'s own history, not
  something this fork's docs should assume has happened.

  Usage by crate (import-site count in `gdna-matrix`'s own source, not
  counting this fork's own internals): `ruma_common` 99, `ruma_signatures`
  20, `ruma_state_res` 11, `ruma_events` 4, `ruma_client_api` 2,
  `ruma_federation_api` 0 (declared as a dependency, no call sites yet).
  `gdna-matrix/src/model.rs` re-exports this fork's identifier and event
  types as the homeserver's own model layer, rather than wrapping them —
  so a breaking change in `ruma-common`'s public API is a breaking change
  in `gdna-matrix`'s model layer directly, with no adapter shim between
  them to absorb it.
- **`gdna-matrix-client`**: this is a **TypeScript/React** web client
  (Vite, already depends on the upstream `matrix-js-sdk` npm package for
  protocol handling), not a Rust project. It cannot consume this Rust SDK
  as a path or git dependency — there is no FFI/WASM binding layer between
  them, and none was scoped for this fork.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Minimum Rust version

Ruma currently requires Rust 1.89. In general, we will never require beta or
nightly for crates.io releases of our crates, and we will try to avoid releasing
crates that depend on features that were only just stabilized.

`ruma-signatures` is an exception: It uses cryptographic libraries that often
use relatively new features and that we don't want to use outdated versions of.
It is guaranteed to work with whatever is the latest stable version though.

## License

[MIT](https://opensource.org/licenses/MIT)
