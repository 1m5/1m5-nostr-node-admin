# 1M5 Nostr Node Admin — TODO

**Do not start Phase 0.** This repository exists to hold the design, not to
be built yet. `1m5-nostr-node-java/TODO.md` ("Explicitly not on any phase
above") gates this on a real operator other than the person running the
relay by hand actually existing, and on that relay's own Phase 4 (NIP-86
management API) shipping first — there is nothing to administer yet.
Rationale is `DESIGN.md`; this file only records what the build order would
be once both preconditions clear, so the next session doesn't have to
re-derive it.

**Preconditions, both required, neither met as of 2026-09-18:**

1. `1m5-nostr-node-java` Phase 4 (NIP-86 + NIP-98) is implemented and
   stable enough that this app isn't developed against a moving target.
2. A real operator other than the person running `psql`-equivalent tooling
   by hand exists, per `1m5-nostr-node-java/TODO.md`'s stated gate.

---

## Phase 0 — Foundations (once gated)

- [ ] `git init`, `LICENSE` (MIT) — already done for this repo.
- [ ] Vite + React + TypeScript scaffold, matching `meridian-admin`'s
      `package.json` baseline (`@tanstack/react-query`, `react-router-dom`,
      `zod`; `vitest`, `@testing-library/react`, `jsdom` for tests) —
      re-check `meridian-admin/package.json` for current versions at build
      time rather than copying pinned numbers from this doc.
- [ ] Confirm `1m5-nostr-node-java`'s actual NIP-86 method names and
      parameter shapes against its live source, not against `DESIGN.md`
      §5's deliberately-unspecified placeholders.

## Phase 1 — Auth (`DESIGN.md` §2)

- [ ] NIP-07 provider detection (`window.nostr` presence) with a clear
      "install a signer" state when absent — not a broken login form.
- [ ] A signed-request wrapper: build the NIP-98 kind-`27235` event (url,
      method, body-hash tag per spec), call `window.nostr.signEvent(...)`,
      attach as `Authorization: Nostr <base64>`. One wrapper, used by every
      authenticated call in later phases — do not hand-roll this per
      feature.
- [ ] Handle the relay rejecting a signature (wrong pubkey, expired
      timestamp per NIP-98) as a distinct, clear UI state from "no NIP-07
      provider at all."

## Phase 2 — Dashboard (`DESIGN.md` §3)

- [ ] NIP-11 fetch and render: which NIPs the node claims, operator
      contact (`npub`).
- [ ] I2P/Tor reachability display, from whatever `1m5-nostr-node-java`'s
      admin surface exposes for `readyTransports()` — confirm the actual
      exposed shape against that repo, not assumed here.

## Phase 3 — Pubkeys (`DESIGN.md` §3)

- [ ] List current ban/allow entries via NIP-86.
- [ ] Add/remove a pubkey, with the reason field if NIP-86 carries one.
- [ ] Optimistic-update or refetch-on-mutate via TanStack Query, consistent
      with whichever pattern `meridian-admin` already uses for its own
      mutations.

## Phase 4 — Storage (`DESIGN.md` §3)

- [ ] Render whatever read-only stats NIP-86 exposes (event counts by kind
      range, disk usage) — no client-side computation of anything the API
      doesn't already return.
- [ ] NIP-45 event-count widget, only if `1m5-nostr-node-java` has shipped
      it by this point (it's SHOULD, not MUST, there) — skip cleanly, not
      with a broken call, if it hasn't.

## Phase 5 — Moderation (`DESIGN.md` §3)

- [ ] Whatever moderation actions NIP-86 actually defines beyond ban/allow
      (e.g., delete-event-by-id) — implement only what the live NIP and
      `1m5-nostr-node-java`'s implementation support; do not add a UI
      affordance for an action the API doesn't have.

## Explicitly not on any phase above

- **Multi-admin pubkey allow-lists / a "who else can administer this"
  view.** Not needed until `1m5-nostr-node-java`'s own identity model
  grows beyond a single operator pubkey (`DESIGN.md` §5).
- **Audit history, RBAC, or any multi-tenant admin workflow.** Those come
  from `meridian-admin` administering a multi-tenant product; this app
  administers one relay for one operator and doesn't need them.
- **Bundling or recommending a specific NIP-07 extension.** This app
  assumes the operator already runs one for their own Nostr identity.