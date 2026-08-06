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

