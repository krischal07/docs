---
title: Troubleshooting
description: Diagnose connect validation, webhook authentication, payload, and delivery issues during integration.
sidebarTitle: Troubleshooting
---



<Info>
**Also see — Purchase QR failures** (`pos_not_connected`, no active communication provider, already processed, expired, QR generation failed): [Request / Response Contract](/integrations/pos/purchase-qr/request-response).
</Info>

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
| `409 pos_not_connected` on purchase QR | The POS integration (RestroX/Blanxer) is not `CONNECTED`/`ACTIVE` (e.g. status `CREATED`) | Confirm the integration is connected in the Samparka dashboard and the token belongs to that integration. | Connect the POS integration, then retry the `purchase-qr` call. |
| `422 No active connected communication provider is configured` on purchase QR | The POS is connected but no WhatsApp/Wapio communication provider is active for the store | Confirm a communication provider is configured and active for the store. | Activate/connect the communication provider, then retry. |
| `409 This bill has already been processed` on purchase QR | The same `bill_id` was previously completed/claimed/failed | Check the session status for the bill. | Use a new `bill_id` for a new sale; a completed bill cannot get a fresh QR. |
| `410 This bill's checkout has expired` on purchase QR | The same `bill_id` previously expired | Check the session expiry. | Use a new `bill_id` for a new sale. |
| `502 QR link could not be generated` on purchase QR | Session created but no short URL (Wapio/redirect failure) | Retry the same `bill_id`. If it repeats, check communication provider health. | Retry; if persistent, contact Samparka. |
| `400 Invalid purchase QR request` on purchase QR | Missing/empty `bill_id`, non-positive `amount`, invalid `customer_phone`, or bad `currency` | Inspect the request body against the [Payload Reference](./payload-reference). | Correct the fields and resend. |
