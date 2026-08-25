# Dogecoin TAP Indexer

The Dogecoin TAP Indexer gives Universe applications a stable, normalized view
of TAP token activity. It reads the canonical Dogecoin mainnet TAP source,
preserves source evidence, and publishes deterministic assets, events, holder
snapshots, and cursor-based journal batches.

## What it provides

- TAP deployments, including canonical Unicode tickers
- mint, transfer, and filled-trade activity
- atomic-unit amounts without floating-point conversion
- holder snapshots published only after a complete generation is verified
- stable event identities and replay-safe journal cursors
- explicit readiness and source-coverage reporting

The service never guesses missing deployment metadata. A malformed or
unresolvable source row is retained as evidence and readiness remains closed
until canonical data is complete.

## Availability model

`GET /live` confirms that the process is running. `GET /ready` confirms that the
pinned reader is connected, the initial scan and holder queue are complete,
there are no quarantined canonical rows, and the latest successful sync is
fresh.

Applications should route traffic only when `/ready` returns HTTP 200. During a
first deployment or source recovery it returns HTTP 503 while durable
checkpoints continue advancing.

## Marketplace readiness

The indexer includes a fail-closed Dogecoin Marketplace integration for TAP on
Dogecoin, DRC-20, and Doginals. It can normalize authority checkpoints, keep
marketplace state replay-safe, prepare transaction intents, validate signed
transactions, and submit a previously authorized transaction to Dogecoin Core.

Marketplace availability is separate from explorer readiness. A protocol is
offered to users only after its canonical authority, fee policy, Core RPC path,
and end-to-end canaries all pass. Until then, marketplace readiness returns an
explicit unavailable result; explorer and journal APIs continue operating.

On a same-host deployment, the authority can read Dogecoin Core authentication
from the node's private configuration file instead of duplicating it in the
service environment. This mode works only over a loopback RPC URL and reloads
the node's current credentials for every request while rejecting links,
oversized files, duplicate credential fields, and files that change during a
read.

Signing remains in the user's wallet. The indexer does not hold wallet keys and
does not silently sign or broadcast transactions.

### Canonical TAP inventory

The optional TAP authority builder produces a complete transferable-inventory
checkpoint only when the pinned TAP reader, Dogecoin Core 1.14.9, and the
official Ord-Dogecoin index agree. Every published asset includes its current
unspent output, value, locking script, raw previous transaction, owner,
inscription satpoint, TAP amount, and deterministic authority identity.
The reader's cumulative transferable balance must cover each individual
transfer inscription amount; a larger cumulative balance is valid, while an
underflow fails the authority build.

The checkpoint is replaced atomically and served only after the same strict
schema used by marketplace ingestion accepts it. A chain-tip change, spent
output, ambiguous inscription output, stale reader, malformed source row, or
source disagreement aborts publication. The previous verified checkpoint is
not silently relabelled as current.

Long-lived immutable transaction and inscription evidence is collected before
the checkpoint window. The final complete UTXO inventory and inscription
custody are then revalidated in bounded batches while the Core tip, Ord tip,
and reader witness are held stable, keeping publication both practical and
fail-closed on Dogecoin's short block interval.

TAP, DRC-20, and Doginals each have a dedicated complete canonical inventory
builder and independent operational gate. DRC-20 and Doginals require a fresh
full Ord-Dogecoin index with their protocol modules enabled; the legacy plain-
inscription archive cannot authorize them. Readiness for one protocol never
enables another indirectly.

## Release validation

Trusted release checks run on the organisation-managed build fleet. A short
infrastructure interruption can receive one automatic retry only when no build
or test step has failed. A real validation failure remains visible and blocks
release promotion.

## Security

Explorer data and status endpoints require Bearer authentication. Reader
compatibility routes are optional, TLS-protected, and restricted to an exact IP
allowlist. The bundled reader REST and WebSocket servers remain disabled.

No wallet seed, private key, signing secret, RPC credential, or database
credential is stored in this documentation.

## Integration

See [API.md](API.md) for the public endpoint contract and readiness behavior.

## Source guarantees

The current reader exposes confirmed indexed state but no Dogecoin mempool feed
or explicit block-replacement notification stream. Coverage therefore reports
`partial`: confirmed activity is canonical at the pinned reader version, while
pending activity is unavailable and reorg handling is based on immutable tail
comparison evidence.

## Universe-operated Wallet data

The indexer now supplies the Wallet's transaction-critical Dogecoin inputs
without public data providers. Its private authority returns proof-bound
confirmed-cardinal wallet summaries and funding plans, excludes protocol assets
and reservations, and binds every response to live Dogecoin Core and the exact
Ord-Dogecoin indexed checkpoint. Backend APIs independently revalidates those
proofs before exposing the bounded public Wallet contract.

## Dogecoin marketplace availability

Browsing and trading are reported as separate facts, because they answer
different questions and can be true independently.

A Doginals or DRC-20 browse surface is available when the indexer holds a
fresh, complete inventory checkpoint that agrees with the Universe-operated
Dogecoin node. Trading additionally requires the execution and broadcast
capabilities to be enabled. Turning trading off never takes browsing offline,
and a capability that is switched off is never presented as available.

Readiness reports `readReady` for browsing, `executionReady` and
`broadcastReady` for trading, and a machine-readable reason whenever a surface
is not available. An application can therefore say "the index is healthy and
this market currently has no listings" instead of the misleading "no tokens
found", and can tell a genuine outage apart from a genuinely empty result.

## Doginals corpus size

The size of a single published Doginals inventory snapshot is a deliberate
operational limit rather than a fixed product ceiling. When a corpus exceeds
the configured limit the indexer reports the observed size and the limit it
crossed, so the decision to raise it can be made alongside the storage and
refresh budget it implies. Inventory is paged with progress reporting, and a
chain tip that advances mid-build retires only that attempt: the last complete
published snapshot stays readable throughout.

## Doginals inventory at production scale

Inventory is read as a stream rather than collected in memory, so the size of
the Dogecoin inscription corpus does not determine what the service needs to
run. Memory stays bounded by the page size and by the marketplace inventory
actually published, and progress is reported per page so a long pass is
visibly advancing rather than silently stalled.

A published inventory snapshot is marketplace custody evidence, not a copy of
the whole corpus. Its size limit is therefore an operational decision, and
when a corpus exceeds it the service reports the observed size and the limit
it crossed instead of failing anonymously. The chain tip is verified before a
pass begins, so a mismatch is reported immediately rather than after a full
scan, and the last complete published snapshot stays readable throughout.

## The service proves what it is connected to

Before serving, the indexer confirms that its configured Dogecoin inscription
index really is a Dogecoin index and reports which protocol data that index
can answer. A misconfigured connection to another chain's index, or to an
index built without Dogecoin token support, is refused at startup with a
message naming what was found.

This matters because an index that was never built with token support answers
token queries with an empty result, which is indistinguishable from a chain
that genuinely has no tokens. The service reports the difference rather than
presenting missing data as an empty market.
