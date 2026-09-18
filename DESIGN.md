# 1M5 Nostr Node Admin — Design

## Status

**Design only, gated.** Written 2026-09-18, the same day `1m5-nostr-node-
java`'s own design was written and this repo was split off from it as a
deferred idea (`1m5-nostr-node-java/TODO.md` "Explicitly not on any phase
above"). That gate — a real operator other than the person running the
relay by hand — has not cleared, and nothing here should be scaffolded
until it does. This document exists so the "what" and "how" are settled in
advance, not so Phase 0 can start now.

Normative keywords (MUST, SHOULD, MAY) are used in the RFC 2119 sense.

## 0. Scope

This document specifies:

1. What this app talks to, and what it deliberately does not reimplement
   (§1) — the NIP-86/NIP-98 surface belongs entirely to `1m5-nostr-node-
   java`.
2. How an operator authenticates (§2) — the one place this app's shape has
   to differ from `meridian-admin`'s, and why.
3. The feature areas a v1 actually needs (§3), scoped to what `1m5-nostr-
   node-java/DESIGN.md` §7 already promises as an admin surface — nothing
   speculative.
4. The frontend stack (§4), decided ahead of the build gate.

### Explicitly out of scope

- **Relay logic, storage, moderation policy.** All of it lives in `1m5-
  nostr-node-java`. This app calls NIP-86 methods; it does not decide what
  "banned" or "over quota" means.
- **A generic multi-relay admin console.** See README.md — incidental
  NIP-86 compatibility with a third-party relay is fine; building for that
  as a goal is not.
- **Any Nostr client capability** (posting notes, following, DMs). That's
  `1m5-core-java`'s `NostrService`, a different role entirely — this app
  never needs to publish an ordinary Nostr event, only NIP-98 auth events
  and NIP-86 management calls.

## 1. What this app talks to

`1m5-nostr-node-java/DESIGN.md` §7 defines the surface this app is a UI
over:

- **NIP-86** (relay management) — ban/allow pubkeys, storage stats,
  moderation actions. Confirm the exact method/parameter shape against the
  live NIP text and against `1m5-nostr-node-java`'s actual implementation
  when that repo's Phase 4 lands — not against this document, which does
  not re-derive the wire format.
- **NIP-11** (relay information document) — read-only, for a status page:
  which NIPs the node claims, and its 1M5-reachability extension field.
- **`CoreClient.readyTransports()`**, surfaced by `1m5-nostr-node-java`
  through its own admin surface (not called directly by this app) — I2P/Tor
  up-or-down, for the reachability panel §3 describes.
- **NIP-45** (event counts), if and once `1m5-nostr-node-java` ships it
  (SHOULD, not MUST there) — an optional stats widget, not a hard
  dependency of this app's v1.

This app holds no relay logic of its own. Every number or action it shows
is a direct read or call against the node's own API; it never computes
storage class, verifies a signature, or decides what counts as spam.

## 2. Authentication: NIP-98 via a signing extension, not a session cookie

**This is the one place this app's shape must differ from `meridian-
admin`**, and it's worth being explicit about why: `meridian-admin`'s
backend (`meridian-backend`) is an ordinary REST API with its own user/
session database, so a backend-managed session cookie is the right model.
This app's "backend" is a Nostr relay — it has no user database, and its
only notion of admin identity is *"a NIP-98-signed HTTP request from the
node operator's own pubkey"* (`1m5-nostr-node-java/DESIGN.md` §6-§7:
"authenticated with this node's own identity, not a separate admin
credential").

Consequences for this app:

- **No login form, no password, no backend session.** The operator's
  identity is a Nostr keypair they already hold, the same way any Nostr
  client authenticates them.
- **Signing happens via a NIP-07 browser extension** (`window.nostr`) —
  the standard way a Nostr web app gets a user's signature without ever
  holding their secret key itself. This app never asks for or stores an
  `nsec`.
- **Every mutating (and, per NIP-98, possibly every) API call is wrapped**
  in a signed kind-`27235` event carrying the request's URL, method, and
  (per spec) a body-hash tag, attached as an `Authorization: Nostr <base64>`
  header. This is a per-request signature, not a session token — there is
  no "logged in" state to persist beyond "a NIP-07 provider is present and
  the operator approved the first prompt."
- **Authorization is enforced by the relay, not this app** (same principle
  `meridian-admin/DESIGN.md` states for its own backend): `1m5-nostr-node-
  java` MUST reject any NIP-98 event whose pubkey isn't its configured
  operator identity (or, later, an allow-listed admin pubkey — not a v1
  concern). This app hiding an action a denied pubkey couldn't perform
  anyway is a UX nicety, never the actual control.
- **No NIP-07 extension present → a clear "install a Nostr signer" state**,
  not a broken login form. This app assumes the operator already runs one
  for their own Nostr identity; it doesn't bundle or recommend a specific
  extension.

## 3. Feature areas (v1)

Scoped tightly to what §1 actually promises — no feature here that doesn't
map to a real NIP-86/NIP-11/`readyTransports()` call:

- **Dashboard** — 1M5 reachability (I2P up/down, Tor up/down, from
  `readyTransports()`), and the relay's own NIP-11 claims (which NIPs, node
  npub for operator contact).
- **Pubkeys** — the ban/allow list NIP-86 exposes: view current entries,
  add/remove a pubkey, with the reason if NIP-86 carries one.
- **Storage** — whatever stats NIP-86 exposes (event counts by kind range
  if available, disk usage if available) — read-only in v1; this app does
  not initiate deletion or compaction itself.
- **Moderation** — whatever moderation actions NIP-86 defines beyond
  ban/allow (e.g., deleting a specific event by id) — only what the live
  NIP and `1m5-nostr-node-java`'s implementation actually support; do not
  invent an action NIP-86 doesn't have just to fill out this section.

Anything not backed by a real API call (audit history, RBAC, multi-admin
workflows — the parts of `meridian-admin`'s scope that come from it
administering a multi-tenant product) is out of scope here. A single-relay
admin panel for a single operator does not need them.

## 4. Frontend stack

**Decided (2026-09-18): same as `meridian/meridian-admin`** — React +
TypeScript + Vite SPA, `@tanstack/react-query` for API state, `react-
router-dom` for routing, `zod` for validating API responses, `vitest` +
`@testing-library/react` for tests. Reusing an org-wide convention beats
picking a bespoke stack for a small single-purpose panel; the only
deliberate divergence from `meridian-admin` is §2's auth model, forced by
the backend being a relay rather than a REST service with its own session
store.

- **API client** — a `fetch` wrapper that, for calls requiring auth, asks
  `window.nostr.signEvent(...)` for a fresh NIP-98 event and attaches it as
  the `Authorization` header; TanStack Query wraps that wrapper the same
  way `meridian-admin`'s API client wraps its own `fetch` calls.
- **Routing** — a handful of routes (`/`, `/pubkeys`, `/storage`,
  `/moderation`), no auth-guard redirect-to-login route, since §2 has no
  login route — the guard state is "no NIP-07 provider" or "provider
  present but the relay rejected the signature," not "not logged in."
- **State management** — server state via TanStack Query, as `meridian-
  admin` does; no separate client-side auth-session store, since there is
  no session (§2).

## 5. Open questions (not decided here)

- **Exact NIP-86 method/parameter shapes.** Deferred to `1m5-nostr-node-
  java`'s own Phase 4 implementation and the live NIP text at that time;
  this document intentionally does not guess a wire format that could
  drift from either.
- **Multi-admin pubkey allow-lists.** §2 assumes a single operator pubkey,
  matching `1m5-nostr-node-java/DESIGN.md`'s current "this node's own
  identity, not a separate admin credential." If that relay-side design
  grows an allow-list, this app's auth model doesn't change (still NIP-98
  per request) but the dashboard could gain a "who else can administer
  this" view — not needed until the relay side has it.

## Definition of Done for this document

This document is done when someone reading only README.md + this file can
answer: what does this app do (§1, §3); why does its auth model look
different from `meridian-admin`'s despite sharing its frontend stack (§2,
§4); and what is genuinely still undecided (§5) versus already settled.
TODO.md turns this into a build order, gated on README.md's stated
precondition.