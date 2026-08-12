---
name: Pull LiveIntent reporting data
description: >-
  Authenticate to the LiveIntent Reporting API and run a metrics query with
  splits, filters and an absolute or dynamic date interval.
api: null
base_url: https://connect.liveintent.com
generated: '2026-08-12'
method: generated
source: https://support.liveintent.com/connecting-to-liveintents-reporting-api/
operations:
  - POST /auth/login/
  - POST /reporting/api/executeQuery
---

# Pull LiveIntent reporting data

> **No machine-readable contract.** Unlike the Audiences and Privacy APIs,
> LiveIntent publishes **no OpenAPI** for the Reporting API. Every field below
> comes from the provider's own knowledge-base article, cited in the
> frontmatter. There is nothing to validate a payload against — build
> defensively.

## Before you start

- **Base URL:** `https://connect.liveintent.com`
  (`api.liveintent.com` resolves to the same address and answers `401` to every
  anonymous path, including `/.well-known/*`).
- You need a **LiveIntent platform username and password** — the same
  credentials you use for `platform.liveintent.com`.

## Step 1 — Get an access token

```
POST https://connect.liveintent.com/auth/login/
Content-Type: application/json

{ "username": "...", "password": "..." }
```

Returns `token`, `username`, `userID` and `refreshToken`.

- The token is valid for **12 hours**, after which it is revoked.
- Cache it for its lifetime. Do not log in per request.
- Store the credentials as secrets — this endpoint takes a **password**, not a
  scoped API key, so the credential is as powerful as the user account itself.

LiveIntent's docs describe this as "OAuth2". It is not: there is no authorize
endpoint, no `client_id`, no scopes and no discovery document. Treat it as a
password login that returns a bearer token.

## Step 2 — Run a query

```
POST https://connect.liveintent.com/reporting/api/executeQuery
Authorization: Bearer {token}
Content-Type: application/json
```

**The path is case sensitive** — `executeQuery`, not `executequery`.

Body:

```json
{
  "type": "publisher",
  "interval": { "type": "absolute", "start": "YYYY-MM-DD", "end": "YYYY-MM-DD" },
  "granularity": "day",
  "splits": [],
  "metrics": [],
  "filters": [
    { "dimension": "", "operator": "equals", "values": [] }
  ]
}
```

| Field | Published values |
|---|---|
| `type` | `publisher`, `advertiser` |
| `granularity` | `day`, `week`, `month`, `all` |
| `interval.type` | `absolute` (with `start`/`end`) or `dynamic` |
| `interval.value` (dynamic) | `WTD`, `MTD`, `QTD`, `YTD`, `1`, `7`, `30`, `90`, `LM` |
| `filters[].operator` | `equals`, `not equals` |

`splits` and `metrics` are string arrays. The knowledge base does not publish an
enumerated list of valid metric or split names — discover them from your account
team or the platform UI. **Do not guess metric names**; an unknown metric is a
silent source of wrong numbers.

## Step 3 — Read the response

An array of objects shaped:

```json
{ "version": "v1", "timestamp": "<ISO-8601>", "event": { } }
```

Each `event` object carries the requested metric and split values as keys.

## Error handling

- **`401 Unauthorized`** — the `Authorization` header is missing, or the token
  is invalid or past its 12-hour expiry. Re-authenticate and retry once.
- The Reporting API does **not** use the `ApplicationError` JSON envelope the
  Audiences and Privacy APIs use. An unauthenticated call to
  `/reporting/api/executeQuery` returns a bare `text/plain` body reading
  `Unauthorized` (observed 2026-08-12). **You cannot parse a machine-readable
  reason from a Reporting API failure** — branch on the status code alone.
- No other status codes are documented. No pagination is documented. No rate
  limits are documented and no `Retry-After` or `X-RateLimit-*` header is
  published.

## Watch for

- Large intervals at `day` granularity with several splits will return large
  arrays; there is no documented paging, so bound your queries by interval
  rather than expecting a cursor.
- The `Reporting API` and `Scheduled Reports` are separately monitored
  components on `https://status.liveintent.com` — check it before assuming a
  query failure is your payload.

## Related

- `authentication/liveintent-authentication.yml` — the full auth profile
- `lifecycle/liveintent-lifecycle.yml` — status page and monitored components
- `rate-limits/liveintent-rate-limits.yml` — why there is nothing to back off on
