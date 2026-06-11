---
title: Troubleshooting
description: Diagnose webhook authentication, payload, mapping, and delivery issues during integration.
sidebarTitle: Troubleshooting
---

| Problem | Likely Cause | Verification Steps | Resolution |
| ------- | ------------ | ------------------ | ---------- |
| `404 Invalid webhook token` | Wrong token in the URL path | Compare the configured URL with the token provided by Samparka. | Update the webhook URL and retry. |
| `400 Request body must be a JSON object` | The request body is missing, malformed, or not a JSON object | Check `Content-Type`, body encoding, and confirm the top-level payload is an object. | Correct the body and resend. |
| `400 Missing event_type in payload` | `event_type` and `type` are both missing | Inspect the JSON body and confirm that one event type field is present. | Add `event_type` and resend. |
| `200 Event already processed` | The same payload was already delivered | Compare the repeated request body with the original request. | Treat the duplicate as success and stop retrying it. |
| `200 Event received` but loyalty missing — no location field sent | `restaurantId` (or alias) was absent from the payload | Confirm the payload includes `restaurantId`, `external_location_id`, `location_id`, `outlet_id`, or `branch_id`. | Add the location identifier and resend. |
| `200 Event received` but loyalty missing — unknown location | The location identifier does not match any mapped location for this integration | Confirm the `restaurantId` value matches the one provided during location sync. | Correct the identifier or resync locations. |
| `200 Event received` but loyalty missing — stale location | The Samparka outlet was removed from the store after the location was synced | Check whether the outlet exists in the merchant's Samparka account. | The merchant should add the outlet back and resync, or the location must be remapped. |
| `200 Event received` but loyalty missing — participation disabled | Location participation was turned off in Samparka | Confirm location participation status with Samparka. | Contact Samparka to re-enable participation. |
| `200 Event received` but loyalty missing — refund not matched | Refund event does not reference the same sale identifier as the original sale | Confirm the refund uses the same `order_id`, `transaction_id`, or `id` that was in the original sale payload. | Resend the refund with the correct original identifier. |
| `200 Event received` but loyalty missing — customer phone | Customer phone was absent or invalid | Confirm `customer.phone`, `phone`, or `customer_phone` is present and valid. | Add a valid phone and resend. |
| `500 Internal server error` | Unexpected server-side failure | Wait and retry the same payload later. | Retry later. If the issue repeats, contact Samparka. |
