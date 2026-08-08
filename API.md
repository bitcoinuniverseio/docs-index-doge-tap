# API

## Health and readiness

### `GET /live`

Public liveness probe. HTTP 200 identifies the `doge-tap` protocol and includes
an observation timestamp.

### `GET /ready`

Public traffic gate. HTTP 200 means all required readiness conditions pass. HTTP
503 is expected during initial backfill, source disconnection, stale sync state,
or canonical quarantine.

Important fields include:

- `initialScanComplete`
- `readerReady`
- `readerConnected`
- `canonicalRowsComplete`
- `quarantinedCanonicalRows`
- `stale`
- `lastSuccessAt`

## Authenticated explorer routes

### `GET /status`

Returns source identity, reader state, durable stream checkpoints, row counts,
coverage, and the latest sync result. It requires a Bearer token.

### `GET /token-explorer/doge-tap`

Returns cursor-based normalized journal batches. Supported query parameters are
`cursor` and `limit`. A cursor is integrity-protected and bound to the configured
source identity. The route returns HTTP 503 until the service is ready.

## Reader compatibility

When explicitly enabled for a trusted backend, `/reader` exposes a narrow
allowlist of legacy read-only TAP operations. It is not a public browser API.
The route requires TLS, Bearer authentication, an exact source-IP match, and a
ready canonical index.

## Dogecoin Marketplace API

Marketplace routes use `/marketplace/v1/{protocol}` where `protocol` is
`tap_doge`, `drc20`, or `doginals`. The unauthenticated readiness route is:

- `GET /marketplace/v1/{protocol}/readiness`

Authenticated routes cover checkpoints, assets and history, listings,
reservations, purchase preparation, offers, wallet challenges, transaction
intents, signed-transaction validation, broadcast, settlements, and explicit
authority synchronization. The machine-readable contract is available at
`GET /marketplace/v1/openapi.json`.

HTTP 503 from protocol readiness is an intentional safety gate. Clients must
not expose listing or transaction actions until readiness is HTTP 200. Preparing
an intent never signs for the user: the wallet reviews and authorizes the exact
transaction before signed validation and broadcast can proceed.

### Canonical authority checkpoint

When the authority publisher is enabled, trusted marketplace consumers can use:

`GET /v1/marketplace/checkpoint?protocol=tap_doge&network=dogecoin-mainnet`

The request requires the configured per-protocol Bearer token and accepts no
body or additional query fields. A valid response contains the canonical
checkpoint, inventory hash, and complete array of transferable assets. The
route returns HTTP 401 for failed authentication, HTTP 404 when that protocol
is not published, and HTTP 503 when no currently valid snapshot can be read.

The same process can ingest its locally published checkpoint without a public
DNS round trip. Local ingestion applies the identical schema and reader-state
validation; it is not a bypass.

## Client behavior

Clients should:

1. Require HTTP 200 from `/ready` before enabling TAP features.
2. Persist the journal cursor only after a complete batch is accepted.
3. Treat HTTP 401 and HTTP 403 as configuration or access-control failures.
4. Treat HTTP 503 as a temporary fail-closed state and retry with backoff.
5. Display source coverage accurately instead of presenting pending or reorg
   data as available.
6. Treat marketplace readiness independently from explorer readiness and keep
   Dogecoin transaction controls disabled while it returns HTTP 503.

