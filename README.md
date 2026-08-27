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

Add as a path dependency from a sibling Rust project (e.g. a `gdna-matrix`
homeserver crate):

```toml
[dependencies]
ruma-common = { path = "../gdna-matrix-sdk/crates/ruma-common" }
ruma-client-api = { path = "../gdna-matrix-sdk/crates/ruma-client-api" }
ruma-federation-api = { path = "../gdna-matrix-sdk/crates/ruma-federation-api" }
ruma-signatures = { path = "../gdna-matrix-sdk/crates/ruma-signatures" }
ruma-state-res = { path = "../gdna-matrix-sdk/crates/ruma-state-res" }
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

- **`gdna-matrix`** (homeserver, Rust): not yet a Cargo project (spec
  scaffolding only, no `Cargo.toml` as of this writing). Path-dependency
  verification is blocked until that project has a buildable crate to test
  against — this is tracked as a prerequisite in that project's own setup,
  not a defect here.
- **`gdna-matrix-client`**: this is a **TypeScript/React** web client
  (Vite, already depends on the upstream `matrix-js-sdk` npm package for
  protocol handling), not a Rust project. It cannot consume this Rust SDK
  as a path dependency — there is no FFI/WASM binding layer between them,
  and none was scoped for this fork. Verifying it "compiles against
  ruma-common/client-api" is not applicable; flagging this as a plan
  assumption that doesn't match the actual repo, not something to route
  around.

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
