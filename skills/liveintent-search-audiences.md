---
name: Search LiveIntent audiences with the filter tree
description: >-
  Query the LiveIntent Audiences API using its composable boolean filter tree,
  and page results with either the offset (v1) or cursor (v2) search endpoint.
api: openapi/liveintent-audiences-openapi.yml
base_url: https://audiences.liveintent.com
generated: '2026-08-12'
method: generated
source: >-
  openapi/liveintent-audiences-openapi.yml and
  https://audiences.liveintent.com/api-guide
operations:
  - POST /search/audiences/v2
  - POST /search/audiences
  - GET /audiences
---

# Search LiveIntent audiences with the filter tree

> **Operation identifiers.** This API declares no `operationId`. Address
> operations by method plus path.

## Which endpoint to use

| Endpoint | Paging | Use it when |
|---|---|---|
| `POST /search/audiences/v2` | cursor (`next`, `pageSize` default 30) | **Default.** Stable under concurrent writes |
| `POST /search/audiences` | offset (`page`, `pageSize`) | Only if you need a total page count |
| `GET /audiences` | offset (`page`, `pageSize`, `updatedAfter`) | Simple listing with no filter tree |

Both search endpoints take `accountId` as a **query** parameter and the filter
in the request body. Neither is marked deprecated in the spec, but v2 is the
newer contract — prefer it.

## The filter tree

The searchable model is richer than the readable one: 23 of the API's 48
schemas exist to express this tree. Compose it from:

**Boolean nodes** — `and`, `or`, `not`
**Match nodes** — `accountIdMatches`, `nameMatches`, `dataProviderIdMatches`,
`ruleIdMatches`, `hasMetadata`, `status`, `eq`
**Typed value nodes** — `Str`, `Num`, `Bool`, `Arr`, `Obj`, `Null`, `Json`
**Scalar filters** — `IntFilter`, `LongFilter`, `StringFilter`, `MetadataFilter`

Build the tree against the schemas in
`openapi/liveintent-audiences-openapi.yml` rather than from memory — the node
names above are the complete published set, and there are no examples in the
spec to copy from.

`status` filters on `SimpleAudienceStatus`: `Active | Archived | Deleted`.

## Paging with v2

1. `POST /search/audiences/v2?accountId={id}` with your filter tree and an
   optional `pageSize` (defaults to 30; **no maximum is documented**).
2. The response is a `CursorPage`: `{ items: [AudienceData], next: string|null }`.
3. Send `next` back in the following request body. Stop when `next` is `null`.

Do not synthesise a cursor — it is opaque. Do not assume a page size limit
exists; if a large `pageSize` fails, it will fail as a `400`, not a documented
cap.

## Paging with v1

`OffsetPaginatedAudiences` returns `{entries, page, pageSize, pages,
totalCount}`. Use `pages` to bound the loop. Offset paging can skip or repeat
rows if audiences are created while you walk, which is the reason to prefer v2.

## What comes back

`AudienceData` is the light projection: `id`, `name`, `accountId`,
`dataProviderId`, `status`, `metadata`, `createdAt`, `updatedAt`.

The full record — `externalAudienceId`, `iabCategory`, `archived`,
`archivesAt`, `deletesAt`, `replacementId`, `dataProviderPrefix`,
`zetaTaxonomyNodeId`, `additionalAttributes` — is only on
`AudienceWithMetadataDTO`, returned by `GET /audiences/{audienceId}`. Search
gives you ids; fetch individually when you need the full shape.

## Watch for

- **Every id is a bare integer** with no prefix. An `accountId`, a
  `dataProviderId`, an audience `id` and a `ruleId` are indistinguishable by
  value. Keep them typed in your own code.
- **`replacementId`** is nullable and undocumented; when set it points at the
  audience that superseded this one.
- **No rate limits, no `429`.** Page politely.

## Error handling

Same envelope as everything else on this API:
`{"errors":[{"httpStatus","message","errorCode"}]}`. `400` on these endpoints
almost always means a malformed filter-tree node. See
`errors/liveintent-problem-types.yml`.

## Related

- `data-model/liveintent-data-model.yml` — the query layer and entity graph
- `conventions/liveintent-conventions.yml` — pagination and filtering conventions
