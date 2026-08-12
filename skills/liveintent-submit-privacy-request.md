---
name: Submit a LiveIntent data subject request
description: >-
  Submit a GDPR/CCPA opt-out, erasure or access request to the LiveIntent
  Privacy Management API, poll it to completion, and retrieve the access report.
api: openapi/liveintent-privacy-openapi.yml
base_url: https://privacy.liadm.com
generated: '2026-08-12'
method: generated
source: >-
  openapi/liveintent-privacy-openapi.yml and https://privacy.liadm.com/api-guide
operations:
  - POST /data-subject-requests/test
  - POST /data-subject-requests
  - GET /data-subject-requests/{transactionId}
  - GET /data-subject-report/{transactionId}
  - POST /search/data-subject-requests
---

# Submit a LiveIntent data subject request

> **Operation identifiers.** This API declares no `operationId`. Address
> operations by method plus path.

**This skill performs an irreversible privacy action.** `ERASURE` deletes
personal data and `RESTRICT` suppresses collection. Neither can be undone
through the API, and there is **no idempotency key** — a retried submission
creates a second request. Treat every call as human-authorised and log the
returned `transactionId`.

## Before you start

- **Base URL:** `https://privacy.liadm.com` (production). Staging is
  `https://privacy-test.liadm.com` — note the spec records this server as a
  bare hostname with no scheme, so mechanical `servers[]` resolution will break.
- **Auth:** `Authorization: Bearer {token}`. The guide states plainly: "To get
  an access token, contact your account team at LiveIntent." There is no token
  endpoint and no scopes.
- **Know your scope.** Authorization here is by account category, not by scope
  string. `DsrScope` is a `oneOf`:

  | Variant | Fields | Blast radius |
  |---|---|---|
  | `Advertiser` | `id` | that advertiser only |
  | `Publisher` | `id` | that publisher and its related advertisers |
  | `MediaGroup` | `id` | **blanket** across every child publisher and advertiser |
  | `PublisherMediaGroup` | `pid`, `mid` | a publisher within a media group |
  | `Global` | — | industry-wide; authorised third-party agents only |

  Choosing `MediaGroup` or `Global` when you meant `Advertiser` is the highest
  consequence mistake available on this API. Select the narrowest variant that
  satisfies the request.

## Steps

### 1. Dry-run against the test endpoint first

`POST /data-subject-requests/test` takes the same body as the real submission
and validates payload shape and account entitlement **without committing**.
Run it before every new integration and after any scope change.

### 2. Submit

`POST /data-subject-requests` with a `NewDataSubjectRequest`:

```json
{
  "action": "RESTRICT",
  "scope": { "Advertiser": { "id": 0 } },
  "jurisdiction": "EU_PRIVACY",
  "emailHashes": [],
  "liveIntentFpcs": [],
  "submitter": null
}
```

- `action` — `RESTRICT` (opt out of sale), `ERASURE` (delete), `ACCESS`
  (produce a report). `ACCESS` is documented as internal use only.
- `jurisdiction` — `EU_PRIVACY` or `US_PRIVACY`.
- `emailHashes` — hashed emails, the join key into LiveIntent's identity graph.
  Both `emailHashes` and `liveIntentFpcs` default to `[]`, so a request naming
  neither identifies nobody. Set at least one.
- `liveIntentFpcs` — LiveIntent first-party cookie identifiers (DUIDs).
- `submitter` — free-form; logged only, no processing applied. Useful for audit.
- `callback` — **do not set it.** The spec marks it internal use only.

Returns **`202 Accepted`**, not `200`, with a `DataSubjectResponse` carrying
`transactionId`. The request has been queued, not completed.

### 3. Poll to completion

`GET /data-subject-requests/{transactionId}` returns
`DataSubjectRequestWithReport` including `status`. There is no webhook —
polling is the only completion signal. Back off between polls; there are no
published rate limits and no `429` to guide you.

### 4. Retrieve an ACCESS report

`GET /data-subject-report/{transactionId}` responds **`303`** with a redirect to
a signed, expiring download URL (also surfaced as `downloadUrl`). Follow the
redirect promptly — a `403` on that URL means the signature expired, not that
you lack permission. Re-issue by calling the endpoint again.

### 5. Audit

`POST /search/data-subject-requests` returns a
`DataSubjectRequestSearchResult`. No pagination parameters are declared on this
endpoint, so do not assume you can page it.

## Error handling

Envelope: `{"errors":[{"httpStatus","message","errorCode"}]}` — an array, not a
single object. Not RFC 9457.

| Status | Meaning |
|---|---|
| 400 | Invalid request body or missing required fields |
| 401 | Missing bearer token |
| 403 | Invalid or expired bearer token — or an expired report signature |
| 404 | Report not found, **or** "the provided account cannot be used" |
| 500 | Internal server error |

Note the two distinctions this API makes that the Audiences API does not: `401`
means *no* token while `403` means a *bad* token, and a `404` on submission is
an entitlement failure — your token's account category is not permitted to act
for the scope you asked for. Do not retry a `404` on `POST
/data-subject-requests`; fix the scope.

## Avoid the Legacy endpoints

`POST /dsr`, `GET /oath` and `GET /submit` are tagged **Legacy** and documented
as "deprecated and should not be used for new integrations". They are **not**
flagged `deprecated: true` in the OpenAPI, so spec-reading tools will not warn
you. `GET /submit` returns a blank GIF. Use `POST /data-subject-requests`.

## Related

- `cli/liveintent-cli.yml` — LiveIntent's own `li-privacy` CLI for this API
- `sandbox/liveintent-sandbox.yml` — the published `dailyplanet.com` staging account
- `errors/liveintent-problem-types.yml` — full error catalog
