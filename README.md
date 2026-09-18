# 1M5 Nostr Node Admin

A web admin panel for operating a [`1m5-nostr-node-java`](https://github.com/1m5/1m5-nostr-node-java)
relay: ban/allow pubkeys, view storage stats, moderate, and see whether the
node's I2P/Tor reachability is actually up right now — over that node's own
NIP-86 relay-management API.

## Status

**Design only, and deliberately not started yet.** This repository holds
this document set and no source. `1m5-nostr-node-java/TODO.md` ("Explicitly
not on any phase above") gates building this on a real operator other than
the person running the relay by hand actually existing — that gate hasn't
cleared. The tech stack below is decided ahead of that anyway, the same way
`1m5-nostr-node-java`'s own storage/framework choices were settled before
Phase 1 started. See [`DESIGN.md`](DESIGN.md) and [`TODO.md`](TODO.md).

## Why a separate repo, and why this scope

Administering a Nostr relay over its own NIP-86 API is a different job from
being a Nostr client, and a different job from running the relay itself —
each already has its own repo (`1m5-nostr-node-java` is the relay; there is
deliberately no dedicated Nostr client repo, since that capability lives
inside `1m5-core-java`'s `NostrService` — see `1m5-nostr-node-java/TODO.md`).
This repo is the third piece: an operator-facing console, kept separate for
the same discoverability reason `1m5-nostr-node-java/DESIGN.md` gives for
staying out of `1m5-core-java` — an operator finding and running an admin
panel shouldn't first need to learn either codebase's full scope.

**This targets `1m5-nostr-node-java` specifically, not "any Nostr relay."**
NIP-86 is a standard, so nothing stops this panel from working against a
conformant third-party relay too, but that's incidental, not the goal — the
same "no generic reach for its own sake" reasoning that already dropped this
project's earlier multi-language client-port plan and its dedicated-client
idea applies here too. Build for the relay this org actually runs.

## Relationship to sibling repos

- **`1m5-nostr-node-java`** — the relay this panel administers. Owns the
  NIP-86 API surface and the NIP-98 auth this panel authenticates against;
  nothing here duplicates either.
- **`meridian/meridian-admin`** — not a dependency, but the tech-stack
  precedent: same frontend stack (`DESIGN.md` §4), adapted where the backend
  shape genuinely differs (§2 — no session cookie, because the backend is a
  relay, not a REST API with its own auth database).

## License

MIT.