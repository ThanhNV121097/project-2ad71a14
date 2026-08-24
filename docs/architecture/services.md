# Services — hello-word-10

## Scope

Backend serves health and one read-only greeting endpoint. API paths use `/v1/...` without `/api` prefix.

## Shared error envelope

All non-2xx API responses return JSON:

```json
{
  "error": {
    "code": "string",
    "message": "string"
  }
}
```

Rules:

- `code` is stable machine-readable snake_case.
- `message` is generic and safe for users.
- Internal error detail stays in logs.

Common codes:

| HTTP | Code | Message |
|---:|---|---|
| 500 | `internal_error` | `Internal server error` |
| 503 | `service_unavailable` | `Service unavailable` |

## Endpoints

### `GET /healthz`

Readiness endpoint used by compose health check.

Request body: none.

Success response:

- Status: `200 OK`
- Content-Type: `text/plain; charset=utf-8`
- Body: `ok`

Failure response:

- Status: `503 Service Unavailable`
- Body: shared error envelope with `service_unavailable`.

Readiness means:

- Migrations have completed successfully.
- Backend can run `SELECT 1` against PostgreSQL.

### `GET /v1/greeting`

Returns singleton greeting stored in PostgreSQL.

Request body: none.

Success response:

- Status: `200 OK`
- Content-Type: `application/json`

```json
{
  "text": "Hello Word"
}
```

Response fields:

| Field | Type | Required | Notes |
|---|---|---|---|
| `text` | string | yes | Stored `greetings.text` value |

Failure responses:

| Condition | Status | Code | Message |
|---|---:|---|---|
| DB unavailable | 503 | `service_unavailable` | `Service unavailable` |
| Greeting row missing | 500 | `internal_error` | `Internal server error` |
| Unexpected server error | 500 | `internal_error` | `Internal server error` |

## Rejected alternatives

| Alternative | Rejected because |
|---|---|
| `/api/v1/greeting` | Deploy proxy strips `/api`; backend must mount `/v1/greeting` |
| `GET /greeting` | Versionless path makes future incompatible contract changes harder |
| Error string at top level | Envelope keeps errors consistent across health-adjacent APIs |
