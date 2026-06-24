---
title: Troubleshooting
description: Diagnose connect validation, webhook authentication, payload, and delivery issues during integration.
sidebarTitle: Troubleshooting
---


| Problem | Likely Cause | Verification Steps | Resolution |
| ------- | ------------ | ------------------ | ---------- |
| `400 externalLocationId is required` on connect | The request omitted the preferred location field or still relies on stale request examples. | Inspect the JSON body for top-level `externalLocationId`. | Send the connect payload with `integrationKey` and `externalLocationId`. |
| `404 Invalid Integration Key` on connect | The `integrationKey` does not resolve a POS integration | Compare the request key with the current Samparka Integration Key. | Replace the key and retry the connect request. |
| `409 Disconnect the integration before rebinding it to another restaurant` | The integration is already connected to a different location | Compare the requested `externalLocationId` with the current binding. | Disconnect first, then reconnect with the new location. |
| `404 Invalid webhook token` | Wrong token in the URL path | Compare the configured URL with the token provided by Samparka. | Update the webhook URL and retry. |
| `400 Request body must be a JSON object` | The request body is missing, malformed, or not a JSON object | Check `Content-Type`, body encoding, and confirm the top-level payload is an object. | Correct the body and resend. |
| `400 Missing event_type in payload` | `event_type` and `type` are both missing | Inspect the JSON body and confirm that one event type field is present. | Add `event_type` and resend. |
| `200 Event already processed` | The same payload was already delivered | Compare the repeated request body with the original request. | Treat the duplicate as success and stop retrying it. |
| `200 Event received` but expected loyalty activity is missing | The integration is disconnected, missing its bound location, the customer phone is missing, or refund linkage needs review | Confirm the integration is still connected, the bound location exists on the integration, the customer phone is present, and the original sale identifier is correct. | Reconnect or rebind the integration if needed, then resend a valid event. |
| `500 Internal server error` | Unexpected server-side failure | Wait and retry the same payload later. | Retry later. If the issue repeats, contact Samparka. |
