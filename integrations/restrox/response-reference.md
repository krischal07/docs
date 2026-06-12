---
title: Response Reference
description: Reference the webhook acknowledgment and error responses returned by Samparka.
sidebarTitle: Response Reference
---

# Response Reference

These are the partner-visible responses for the canonical RestroX webhook path.
For outlet-owned RestroX, restaurant attribution is resolved from the integration bound to the webhook token.

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

Source:
samparka-backend/src/integrations/pos/controller.js:332-365

## `200 Event already processed`

```json
{
  "success": true,
  "message": "Event already processed"
}
```

Meaning: the same webhook was sent before, and Samparka handled it safely.

Recommended sender behavior: stop retrying the same payload.

Source:
samparka-backend/src/integrations/pos/controller.js:293-312

## `400 Request body must be a JSON object`

```json
{
  "success": false,
  "message": "Request body must be a JSON object"
}
```

Meaning: the request body was missing, not JSON, or was an array instead of an object.

Recommended sender behavior: correct the request body and resend.

Source:
samparka-backend/src/integrations/pos/providers/restrox/validator.js:23-26
samparka-backend/src/integrations/pos/controller.js:217-221

## `400 Missing event_type in payload`

```json
{
  "success": false,
  "message": "Missing event_type in payload"
}
```

Meaning: the payload did not include `event_type` or `type`.

Recommended sender behavior: add the event type and resend.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:20-23
samparka-backend/src/integrations/pos/controller.js:235-243

## `404 Invalid webhook token`

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

Meaning: the token in the URL did not match a configured RestroX webhook location.

Recommended sender behavior: verify the webhook URL with Samparka before retrying.

Source:
samparka-backend/src/integrations/pos/controller.js:200-205

## `500 Internal server error`

```json
{
  "success": false,
  "message": "Internal server error"
}
```

Meaning: Samparka encountered an unexpected error while handling the request.

Recommended sender behavior: retry later with the same payload.

Source:
samparka-backend/src/integrations/pos/controller.js:367-397
