---
title: Response Reference
description: Reference the webhook acknowledgment and error responses returned by Samparka.
sidebarTitle: Response Reference
---

These are the partner-visible responses for the canonical RestroX webhook path.

See also: [Troubleshooting](./troubleshooting).

## `200 Event received`

```json
{
  "success": true,
  "message": "Event received"
}
```

Meaning: Samparka accepted the delivery.

Recommended sender behavior: record the delivery as acknowledged. Do not retry unless you later confirm the request did not reach Samparka.

### What 200 Does Not Mean

HTTP 200 confirms the event was received and accepted. It does not indicate the outcome of loyalty processing. All of the following scenarios return HTTP 200:

| Scenario | Internal Result |
|----------|-----------------|
| Sale processed, loyalty awarded | loyalty earned |
| Sale received, no customer phone supplied | skipped — no_phone |
| Sale received, location not mapped | blocked_unmapped_location |
| Duplicate sale resent | Event already processed |
| Refund received, no matching original sale | failed_terminal — original_event_not_found |

## `200 Event already processed`

```json
{
  "success": true,
  "message": "Event already processed"
}
```

Meaning: the same webhook was sent before, and Samparka handled it safely.

Recommended sender behavior: stop retrying the same payload.

## `400 Request body must be a JSON object`

```json
{
  "success": false,
  "message": "Request body must be a JSON object"
}
```

Meaning: the request body was missing, not JSON, or was an array instead of an object.

Recommended sender behavior: correct the request body and resend.

## `400 Missing event_type in payload`

```json
{
  "success": false,
  "message": "Missing event_type in payload"
}
```

Meaning: the payload did not include `event_type` or `type`.

Recommended sender behavior: add the event type and resend.

## `404 Invalid webhook token`

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

Meaning: the token in the URL did not match a configured RestroX webhook location.

Recommended sender behavior: verify the webhook URL with Samparka before retrying.

## `500 Internal server error`

```json
{
  "success": false,
  "message": "Internal server error"
}
```

Meaning: Samparka encountered an unexpected error while handling the request.

Recommended sender behavior: retry later with the same payload.
