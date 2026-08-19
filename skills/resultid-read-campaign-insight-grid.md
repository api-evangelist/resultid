---
name: Read a Resultid campaign's insight grid
description: Resolve a campaign, read its data and themes, then pull the insight grid rows for it, optionally grouped by a grouping column.
api: openapi/resultid-api-openapi.yml
operations:
  - do_get_campaign_uuids_get_campaign_uuids_get
  - do_get_campaign_data_get_campaign_data_get
  - do_get_campaign_theme_data_get_campaign_theme_data_get
  - do_read_insight_grid_data_read_insight_grid_data_get
  - do_read_campaign_trend_data_read_campaign_trend_data_get
generated: '2026-08-14'
method: generated
source: openapi/resultid-api-openapi.yml
---

# Read a Resultid campaign's insight grid

Resultid's read API is campaign-scoped. Almost everything you can ask for hangs off a
`campaign_uuid`, so the flow is: hold a campaign UUID, describe the campaign, then read the grid.

## Before you start

- **You need an API key.** Every operation requires an `X-API-Key` request header. Resultid does
  not document how a key is issued — ask your Resultid contact.
- **You need a base URL.** Resultid publishes no `servers[]` block and no host on
  docs.resultid.com. Get the base URL from Resultid before writing any client; do not guess it.
- **Every operation is a `GET` with a required JSON request body.** This is unusual. Verify your
  HTTP client actually transmits a body on `GET` — many strip it silently, and you will see a
  `422` with no obvious cause.

## Steps

1. **Resolve the campaign.** Call `do_get_campaign_uuids_get_campaign_uuids_get`
   (`GET /get_campaign_uuids`) with `{"campaign_uuid": "<uuid>"}`. Hold the campaign UUID you
   intend to work with.
2. **Describe the campaign.** Call `do_get_campaign_data_get_campaign_data_get`
   (`GET /get_campaign_data`) with `{"campaign_uuid": "<uuid>"}`.
3. **Read the themes.** Call `do_get_campaign_theme_data_get_campaign_theme_data_get`
   (`GET /get_campaign_theme_data`) with the same body. Themes are the campaign-level topical
   rollup.
4. **Read the insight grid.** Call `do_read_insight_grid_data_read_insight_grid_data_get`
   (`GET /read_insight_grid_data`) with `{"campaign_uuid": "<uuid>"}`. Add
   `"grouping_column_uuid": "<uuid>"` to group the rows; only `campaign_uuid` is required.
5. **Read the trend series, if you need movement over time.** Call
   `do_read_campaign_trend_data_read_campaign_trend_data_get` (`GET /read_campaign_trend_data`)
   with `{"campaign_uuid": "<uuid>"}`.

## Rules

- **Do not paginate.** There is no pagination contract — no limit, offset or cursor parameter and
  no cursor in the response. Treat every grid read as an unbounded full read and be prepared for
  a large payload.
- **Do not retry blindly on a `422`.** `422 Unprocessable Content` means the body failed schema
  validation. Read `detail[].loc` to find the offending field and fix the request; retrying the
  same body will fail identically.
- **Assume nothing about `401`/`403`/`404`/`429`.** Resultid declares none of them. Handle
  non-2xx defensively and log the raw body.
- **Do not assume a response shape.** All nine operations declare an empty `200` schema, so
  response fields are not part of the contract. Parse tolerantly and pin nothing.
- **Reads are safe to repeat.** All nine operations are reads; there is no idempotency key
  because there is no write surface.

## See also

- `conventions/resultid-conventions.yml` — the cross-cutting semantics, and what is missing
- `errors/resultid-problem-types.yml` — the `422` envelope
- `authentication/resultid-authentication.yml` — the `X-API-Key` header
