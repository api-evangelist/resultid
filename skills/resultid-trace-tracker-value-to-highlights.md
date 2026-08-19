---
name: Trace a Resultid tracker value to its source highlights
description: Enumerate the distinct values of a tracker within a campaign, then pull every source highlight attached to one of those values.
api: openapi/resultid-api-openapi.yml
operations:
  - do_get_unique_values_for_tracker_get_unique_values_for_tracker_get
  - do_get_all_highlights_for_tracker_value_get_all_highlights_for_tracker_value_get
  - do_get_insight_last_result_get_insight_last_result_get
generated: '2026-08-14'
method: generated
source: openapi/resultid-api-openapi.yml
---

# Trace a Resultid tracker value to its source highlights

This is the drill-down path: from a tracked dimension, to one of its values, to the underlying
excerpts that produced it. It is the only route in the published API from an aggregate back to
source evidence.

## Before you start

- Every operation requires an `X-API-Key` request header.
- Every operation is a `GET` carrying a required JSON request body.
- You need a `campaign_uuid` and a `tracker_uuid` to begin. Neither is discoverable from a list
  operation in the published contract — hold them from the Resultid web app or from your
  Resultid contact.

## Steps

1. **Enumerate the tracker's values.** Call
   `do_get_unique_values_for_tracker_get_unique_values_for_tracker_get`
   (`GET /get_unique_values_for_tracker`) with
   `{"campaign_uuid": "<uuid>", "tracker_uuid": "<uuid>"}`. Both fields are required.
2. **Pick a value.** The values are **strings**, not UUIDs — `tracker_value_str` is the only
   non-UUID key in the API. Pass the value back exactly as returned; do not normalise case,
   trim, or re-encode it.
3. **Read the highlights.** Call
   `do_get_all_highlights_for_tracker_value_get_all_highlights_for_tracker_value_get`
   (`GET /get_all_highlights_for_tracker_value`) with
   `{"campaign_uuid": "<uuid>", "tracker_value_str": "<value>"}`. Both fields are required.
   Note the campaign is passed again — highlights are scoped to campaign plus value, not to the
   tracker.
4. **Optionally attach the insight.** If you already hold an `insight_uuid`, call
   `do_get_insight_last_result_get_insight_last_result_get` (`GET /get_insight_last_result`)
   with `{"insight_uuid": "<uuid>"}` to pair the most recent computed result with the highlights
   you just read. There is no operation that lists insights, so you cannot discover the UUID
   from the API.

## Rules

- **Do not construct `tracker_value_str` yourself.** Only values returned by step 1 are valid;
  a value you invent will either return nothing or `422`.
- **Do not paginate.** `get_all_highlights_for_tracker_value` returns *all* highlights for the
  value with no limit or cursor parameter. On a high-volume campaign this can be a large
  response.
- **Handle `422` by reading `detail[].loc`.** It names the field that failed validation.
- **Treat every operation as read-only.** Nothing in the published API writes, so re-running any
  step is safe.

## See also

- `data-model/resultid-data-model.yml` — Campaign → Tracker → TrackerValue → Highlight
- `conventions/resultid-conventions.yml` — the GET-with-body shape and what is undocumented
