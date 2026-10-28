# Peerspan (Go)

A Go implementation of the [Peerspan](https://github.com/peerspan/peerspan) host daemon and CLI, plus the optional helper and test tools. Peerspan connects agents across machines and owners.

The main [Peerspan repository](https://github.com/peerspan/peerspan) holds the specification, the Rust host, the agent integrations, and the conformance tests. Peerspan's primary implementation is Rust. This repository continues the Go implementation separately.

## Where this code was taken from

This repository was split from `go/` in [`peerspan/peerspan` at commit `3b42f67`](https://github.com/peerspan/peerspan/tree/3b42f672659609fb4b31d94a2ef9df071fadc7c8) (`3b42f672659609fb4b31d94a2ef9df071fadc7c8`), with `go/`'s history kept. The only changes since the split are the module path, now `github.com/peerspan/peerspan-go`, this README, the licence file, and CI.

At that commit:

- **Specification:** the coordinator-free design is [`docs/decentralized-spec.md`](https://github.com/peerspan/peerspan/blob/3b42f672659609fb4b31d94a2ef9df071fadc7c8/docs/decentralized-spec.md) and [`docs/decentralized-wire-formats.md`](https://github.com/peerspan/peerspan/blob/3b42f672659609fb4b31d94a2ef9df071fadc7c8/docs/decentralized-wire-formats.md), the simplified spec with owner decisions DEC-S1 to DEC-S17 and DEC-Q1 to DEC-Q33. The product behaviour is in the other contracts under [`docs/`](https://github.com/peerspan/peerspan/tree/3b42f672659609fb4b31d94a2ef9df071fadc7c8/docs), and the [decision log](https://github.com/peerspan/peerspan/blob/3b42f672659609fb4b31d94a2ef9df071fadc7c8/docs/decision-log.md) records every owner decision.
- **This code** implements the coordinator-free runtime as first merged (peerspan-internal PR #139), which predates the simplified spec, and so does the Rust host at the same commit. Where the two differ, the spec is the target. Known gaps include: invites and link requests that last 300 seconds instead of the spec's lifetimes; no device grant for the creator's own device (DEC-Q21); and in-place migration code for coordinator networks, which the spec drops (DEC-OWN3).
- **Coordinator:** `cmd/peerspan-coord` and `internal/coord` are the retired coordinator (DEC-OWN2). They are kept because hosts still read existing coordinator networks as read-only history. New coordinator networks are refused.
- **Known Go-specific bugs:** porting Peerspan's product test suites to coordinator-free networks found these. They were not fixed before the split.
  - A Go root administrator can't admit a Rust requester (`state_conflict`).
  - `access block --person` is refused with `upgrade required`.
  - A policy edit whose starting policy is stale fails instead of reloading.
  - The Go daemon refuses a Rust CLI's configured owner key for `allow`.
  - The Go CLI's `attach claude --no-start` against a Rust daemon fails with `use the decentralized network API`.

  The Go and Rust `network persons` outputs also differ on coordinator-free networks.

## Contents

```text
cmd/peerspan-host/        Host daemon and owner CLI
cmd/peerspan-helper/      Optional mailbox and relay helper
cmd/peerspan-peer-probe/  Peer protocol probe used by conformance tests
cmd/peerspan-link-probe/  Linking probe used by conformance tests
cmd/peerspan-coord/       Retired coordinator (legacy history only)
internal/                 Shared packages
```

## Build and test

```sh
go build ./cmd/...
go test -race ./internal/... ./cmd/...
go vet ./internal/... ./cmd/...
```

The black-box conformance suites and the Node.js agent integrations live in the [main repository](https://github.com/peerspan/peerspan). The suites at the split commit take a Go host binary via `--go PATH`.

## License

Apache License 2.0. See [LICENSE](LICENSE).
