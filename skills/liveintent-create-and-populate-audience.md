---
name: Create and populate a LiveIntent custom audience
description: >-
  Create a custom audience on the LiveIntent Audiences API, upload its member
  hashes through a signed URL, poll the upload to completion, and read the
  match counts.
api: openapi/liveintent-audiences-openapi.yml
base_url: https://audiences.liveintent.com
generated: '2026-08-12'
method: generated
source: >-
  openapi/liveintent-audiences-openapi.yml and
  https://audiences.liveintent.com/api-guide
operations:
  - POST /audiences
  - POST /audiences/{audienceId}/signed-urls
  - GET /audiences/{audienceId}/uploads
  - GET /audiences/{audienceId}/counts
  - GET /audiences/{audienceId}
---

# Create and populate a LiveIntent custom audience

> **Operation identifiers.** The LiveIntent Audiences OpenAPI declares no
> `operationId` on any of its 12 operations. Every step below is addressed by
> HTTP method plus path, which is what the contract actually publishes. Do not
> invent operationIds.

## Before you start

- **Base URL:** `https://audiences.liveintent.com` (production).
  The spec lists `https://audiences-staging.bln.liveintent.com` **first** in
  `servers[]`, so a client that blindly takes `servers[0]` will write to
  staging. Choose the server explicitly.
- **Auth:** every request needs `Authorization: Bearer {token}`. There is no
  token endpoint — tokens are issued by the LiveIntent account team. A missing
  token returns `401` with
  `{"errors":[{"httpStatus":401,"message":"Token not provided","errorCode":"unauthorized"}]}`.
- **You will need** an `accountId` and a `dataProviderId`. Neither is
  discoverable through this API; both come from your LiveIntent account setup.
- **There is no idempotency key.** `POST /audiences` is not retry-safe. If it
  times out, search for the audience by name before creating it again.

## Steps

### 1. Create the audience

`POST /audiences` with a `CreateAudienceRequest`:

```json
{ "name": "...", "accountId": 0, "dataProviderId": 0, "metadata": {} }
```

Returns `201` with a `CreateAudienceResponse`. Keep the returned `id` — it is
the `audienceId` for every step that follows. `500` here means
"Failed to create audience"; retry with backoff, then verify by search rather
than blind-retrying (see the no-idempotency warning above).

### 2. Mint a signed upload URL

`POST /audiences/{audienceId}/signed-urls` with a `CreateSignedUrlRequest`.
Returns `SignedUploadUrlResponse` containing `signedUrl`.

The member data does **not** flow through the LiveIntent API host. Upload the
file directly to `signedUrl`. Treat the URL as short-lived and single-purpose;
mint a fresh one per upload rather than caching it.

### 3. Poll the upload to completion

`GET /audiences/{audienceId}/uploads` (paginate with `page` and `pageSize`).
Each `AudienceUpload` carries a `status` from
`Pending | Running | Finished | Error`, plus `uploadedHashesCount`,
`invalidHashesCount` and `estimatedDuplicateCount`.

There is **no webhook and no callback** on this flow — polling is the only
completion signal LiveIntent offers. Set `notificationEmail` on the upload if a
human also needs to be told.

Stop on `Finished`. On `Error`, read `invalidHashesCount` before re-uploading:
a high invalid count usually means the hash format is wrong, not that the
upload failed transiently.

### 4. Read the match counts

`GET /audiences/{audienceId}/counts` returns `AudienceCounts`, a series of
`AudienceCount` entries of `{date, totalCount, matchCount}`.

`matchCount` is the number that matters — it is how many uploaded records
LiveIntent actually resolved. Expect it to lag the upload; counts are produced
daily, so do not treat a low `matchCount` immediately after `Finished` as a
failure.

### 5. Confirm state

`GET /audiences/{audienceId}` returns `AudienceWithMetadataDTO`. Check
`status` is `Active` (the enum is `Active | Archived | Deleted`).

## Error handling

All errors on this API use one envelope:

```json
{"errors":[{"httpStatus":0,"message":"","errorCode":""}]}
```

`errors` is an **array** — read all of them, not just the first. This is not
RFC 9457 problem+json; there is no `type` URI and no `title`.

| Status | Meaning | Action |
|---|---|---|
| 400 | Invalid request parameters | Fix the payload; do not retry unchanged |
| 401 | Token missing or incorrect | Re-issue the token via the account team |
| 404 | Audience not found | Check `audienceId` |
| 500 | Named per-operation failure | Retry with backoff, then verify by search |

**No `429` is declared anywhere in this API and LiveIntent publishes no rate
limits.** There is no `Retry-After` and no `X-RateLimit-*` header to read. Rate
yourself conservatively — you have no runtime signal to back off on.

## Related

- `conventions/liveintent-conventions.yml` — pagination, error envelope, bulk upload
- `errors/liveintent-problem-types.yml` — full error catalog
- `data-model/liveintent-data-model.yml` — Audience, AudienceUpload, AudienceCount
