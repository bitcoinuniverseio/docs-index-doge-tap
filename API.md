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

## Client behavior

Clients should:

1. Require HTTP 200 from `/ready` before enabling TAP features.
2. Persist the journal cursor only after a complete batch is accepted.
3. Treat HTTP 401 and HTTP 403 as configuration or access-control failures.
4. Treat HTTP 503 as a temporary fail-closed state and retry with backoff.
5. Display source coverage accurately instead of presenting pending or reorg
   data as available.

