---
title: Troubleshooting
description: Diagnose webhook authentication, payload, mapping, and delivery issues during integration.
sidebarTitle: Troubleshooting
---

# Troubleshooting

| Problem | Likely Cause | Verification Steps | Resolution |
| ------- | ------------ | ------------------ | ---------- |
| `404 Invalid webhook token` | Wrong token in the URL path | Compare the configured URL with the token provided by Samparka. | Update the webhook URL and retry. |
| `400 Request body must be a JSON object` | The request body is missing, malformed, or not a JSON object | Check `Content-Type`, body encoding, and confirm the top-level payload is an object. | Correct the body and resend. |
| `400 Missing event_type in payload` | `event_type` and `type` are both missing | Inspect the JSON body and confirm that one event type field is present. | Add `event_type` and resend. |
| `200 Event already processed` | The same payload was already delivered | Compare the repeated request body with the original request. | Treat the duplicate as success and stop retrying it. |
| `200 Event received` but expected loyalty activity is missing | The location mapping, customer phone, or refund linkage needs review | Confirm `external_location_id`, customer phone, and original sale identifier. | Correct the data or confirm the outlet mapping with Samparka. |
| `500 Internal server error` | Unexpected server-side failure | Wait and retry the same payload later. | Retry later. If the issue repeats, contact Samparka. |
