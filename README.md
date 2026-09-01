# Dogecoin TAP indexer documentation (archived)

> ## ARCHIVED. FROZEN. NOT MAINTAINED.
>
> **Archived:** 25 August 2026
> **Final version:** none. This repository was continuously published and never carried a release tag. The documentation is frozen at its final source commit.
> **Final source commit:** [`c032045`](https://github.com/bitcoinuniverseio/docs-index-doge-tap/commit/c0320450eea63d7e29f3d62341b07c42a472cbcd) (25 August 2026)
> **Replacement:** [TAP on Doge protocol documentation](https://bitcoinuniverseio.github.io/tap-on-doge/) and the portal's [TAP on Doge page](https://docs.bitcoinuniverse.io/protocols/tap_doge/)
>
> This repository receives no changes, no fixes, and no support. Every
> availability, readiness, and capability statement preserved here describes the
> service as it stood on 25 August 2026, not as it stands today.
>
> Read the [security and accuracy warning](#security-and-accuracy-warning) before you build against anything here.

## What this documented

This repository held the public documentation for the Dogecoin TAP indexer: a
Universe-operated service that read TAP token activity on Dogecoin mainnet and
published it as a normalized, deterministic view for Universe applications.

The documented service covered:

| Area | What it did |
| --- | --- |
| Explorer data | TAP deployments, mints, transfers, and filled trades, in atomic units with no floating-point conversion |
| Journal | Cursor-based batches with stable event identities and replay-safe cursors |
| Holder snapshots | Published only after a complete generation was verified |
| Readiness | `/live` and `/ready` as explicit traffic gates, fail-closed during backfill or source loss |
| Marketplace | A fail-closed Dogecoin marketplace integration for TAP on Doge, DRC-20, and Doginals |
| Coverage reporting | Explicit `partial` coverage: confirmed activity authoritative, pending activity unavailable |

It was documentation for an operated service, not a protocol specification and
not a public API. The service itself was never open source, and its source
repository is private.

## Where this documentation went

Nothing was deleted. The subject matter split into three places that are
maintained, and each of them is more accurate today than anything in this
repository:

| If you came here for | Go to |
| --- | --- |
| The TAP on Dogecoin protocol: specification, guide, indexer semantics, test vectors, payload validator | [bitcoinuniverseio.github.io/tap-on-doge](https://bitcoinuniverseio.github.io/tap-on-doge/) |
| How Universe supports TAP on Doge, and its current status | [docs.bitcoinuniverse.io/protocols/tap_doge/](https://docs.bitcoinuniverse.io/protocols/tap_doge/) |
| The Dogecoin ordinals implementation this indexer depended on (Doginals, DRC-20, Dunes) | [bitcoinuniverseio/ord-dogecoin](https://github.com/bitcoinuniverseio/ord-dogecoin) |
| TAP on Bitcoin mainnet | [bitcoinuniverseio.github.io/tap](https://bitcoinuniverseio.github.io/tap/) |
| Every protocol Universe documents | [Protocol Atlas](https://docs.bitcoinuniverse.io/protocols/) |
| Machine-readable interface contracts across the estate | [Interface directory](https://docs.bitcoinuniverse.io/developers/interfaces/) |

## Migration guidance

**If you consumed the documented HTTP API.** You were a trusted Universe backend
holding a Bearer token; there was never a public consumer of these routes. Get
the current contract from the operator, not from this repository. The routes,
readiness fields, and cursor semantics recorded here are a 2026 snapshot and are
not guaranteed to match anything running now.

**If you were reading this to understand TAP on Dogecoin.** Read
[the TAP on Doge protocol documentation](https://bitcoinuniverseio.github.io/tap-on-doge/)
instead. It is maintained, it carries test vectors and a client-side payload
validator, and it describes the protocol rather than one operator's service.

**If you were reading this for Doginals or DRC-20.** Those were secondary
subjects here. [`ord-dogecoin`](https://github.com/bitcoinuniverseio/ord-dogecoin)
is the Dogecoin ordinals indexer, HTTP API, and explorer, and it is the accurate
source for how Doginals, DRC-20, and Dunes are indexed on Dogecoin.

**If you copied the readiness model.** The shape is still sound and it is worth
reading: separate liveness from readiness, report browse availability separately
from trade availability, and never render "not serving" as "empty". Those rules
are now stated for the whole organization in the
[lifecycle and availability vocabulary](https://docs.bitcoinuniverse.io/status/),
which is the maintained version.

## Security and accuracy warning

The preserved documents were written for an operated service. Some of what they
say is unsafe to act on now.

1. **This is not a public API, and no base URL is published.** Every route beyond
   `/live` and `/ready` required a Bearer token issued to a trusted backend. The
   reader compatibility route additionally required TLS and an exact source-IP
   match. Do not probe, scan, or attempt to reach a Universe-operated instance;
   there is no public endpoint to call and no public token to use.
2. **Do not treat the preserved contract as current.** Route paths, readiness
   field names, cursor formats, and the marketplace protocol list are a snapshot
   from 25 August 2026. Building a client against them without confirming the
   live contract will produce a client that fails silently or, worse, one that
   believes a safety gate is satisfied when it is not.
3. **The readiness gates are safety gates, not health cosmetics.** The preserved
   guidance says a client must keep Dogecoin transaction controls disabled while
   marketplace readiness returns HTTP 503. If you carry that pattern into new
   code, carry the gate with it. A client that shows trade controls over an index
   that is not authoritative can lead a user to sign against stale state, and
   everything on Dogecoin is real and irreversible.
4. **`GET /marketplace/v1/openapi.json` is not publicly reachable.** It is
   referenced in the preserved API document as the machine-readable contract of
   an authenticated service. Published Universe interface contracts are listed in
   the [interface directory](https://docs.bitcoinuniverse.io/developers/interfaces/).
5. **Availability statements here are historical.** They describe what was true
   at archive time. The authority for current availability is
   [live status](https://docs.bitcoinuniverse.io/status/live/), never archived
   prose.

No wallet seed, private key, signing secret, RPC credential, database credential,
hostname, or IP address appears in this repository, and none was removed to make
that true.

## What is preserved here

| Path | What it is |
| --- | --- |
| [`API.md`](API.md) | The final public endpoint and readiness contract, unchanged below its historical banner |
| [`archive/README-final-2026-08-25.md`](archive/README-final-2026-08-25.md) | The README as published on the final source commit, byte-identical |

Provenance, so any copy can be checked against Git history:

| File | Source | SHA-256 |
| --- | --- | --- |
| `API.md` body | `API.md` at `c032045` | `330ea616e209d153dc0689d4519cbfe27b61aa4f7201447d35cc6180ca8d132a` |
| `archive/README-final-2026-08-25.md` | `README.md` at `c032045` | `1a5b71ec1bf15faf242960d857fdd8a72bb2c46c00df53b71936ad9f9b96977f` |

```bash
git show c0320450eea63d7e29f3d62341b07c42a472cbcd:README.md | sha256sum
git show c0320450eea63d7e29f3d62341b07c42a472cbcd:API.md | sha256sum
```

`API.md` keeps its original path so links into it from elsewhere keep resolving.
A historical banner was added at the top; everything below that banner is
unchanged from `c032045`, and the API contract itself last changed on 16 August
2026 in commit
[`7db92da`](https://github.com/bitcoinuniverseio/docs-index-doge-tap/commit/7db92da).

## Permanent URLs

| What | URL |
| --- | --- |
| This repository | https://github.com/bitcoinuniverseio/docs-index-doge-tap |
| Preserved API contract | https://github.com/bitcoinuniverseio/docs-index-doge-tap/blob/main/API.md |
| Final source commit | https://github.com/bitcoinuniverseio/docs-index-doge-tap/commit/c0320450eea63d7e29f3d62341b07c42a472cbcd |
| Replacement protocol documentation | https://bitcoinuniverseio.github.io/tap-on-doge/ |
| Portal page for TAP on Doge | https://docs.bitcoinuniverse.io/protocols/tap_doge/ |
| Dogecoin ordinals implementation | https://github.com/bitcoinuniverseio/ord-dogecoin |
| Documentation home | https://docs.bitcoinuniverse.io |
