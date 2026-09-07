---
title: Troubleshooting
description: Diagnose connect validation, webhook authentication, payload, and delivery issues during integration.
sidebarTitle: Troubleshooting
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
<Info>
**Also see — Purchase QR failures** (`system_not_connected`, no active communication provider, already processed, expired, QR generation failed): [Request / Response Contract](/integrations/system/purchase-qr/request-response).
</Info>

| Problem | Likely Cause | Verification Steps | Resolution |
| ------- | ------------ | ------------------ | ---------- |
| `400 externalLocationId is required` on connect | The request omitted the preferred location field or still relies on stale request examples. | Inspect the JSON body for top-level `externalLocationId`. | Send the connect payload with `integrationKey` and `externalLocationId`. |
| `404 Invalid Integration Key` on connect | The `integrationKey` does not resolve a System integration | Compare the request key with the current Samparka Integration Key. | Replace the key and retry the connect request. |
| `409 Disconnect the integration before rebinding it to another restaurant` | The integration is already connected to a different location | Compare the requested `externalLocationId` with the current binding. | Disconnect first, then reconnect with the new location. |
| `404 Invalid webhook token` | Wrong token in the URL path | Compare the configured URL with the token provided by Samparka. | Update the webhook URL and retry. |
| `400 Request body must be a JSON object` | The request body is missing, malformed, or not a JSON object | Check `Content-Type`, body encoding, and confirm the top-level payload is an object. | Correct the body and resend. |
| `400 Missing event_type in payload` | `event_type` and `type` are both missing | Inspect the JSON body and confirm that one event type field is present. | Add `event_type` and resend. |
| `200 Event already processed` | The same payload was already delivered | Compare the repeated request body with the original request. | Treat the duplicate as success and stop retrying it. |
| `200 Event received` but expected loyalty activity is missing | The integration is disconnected, missing its bound location, the customer phone is missing, or refund linkage needs review | Confirm the integration is still connected, the bound location exists on the integration, the customer phone is present, and the original sale identifier is correct. | Reconnect or rebind the integration if needed, then resend a valid event. |
| `500 Internal server error` | Unexpected server-side failure | Wait and retry the same payload later. | Retry later. If the issue repeats, contact Samparka. |
| `409 system_not_connected` on purchase QR | The System integration is not `CONNECTED`/`ACTIVE` (e.g. status `CREATED`) | Confirm the integration is connected in the Samparka dashboard and the token belongs to that integration. | Connect the System integration, then retry the `purchase-qr` call. |
| `422 No active connected communication provider is configured` on purchase QR | The System is connected but no WhatsApp/Wapio communication provider is active for the store | Confirm a communication provider is configured and active for the store. | Activate/connect the communication provider, then retry. |
| `409 This session has already been processed` on purchase QR | The same request was previously completed/claimed/failed | Check the session status. | Make a new request for a new sale; a completed session cannot get a fresh QR. |
| `410 This session's checkout has expired` on purchase QR | The same request previously expired | Check the session expiry. | Make a new request for a new sale. |
| `502 QR link could not be generated` on purchase QR | Session created but no short URL (Wapio/redirect failure) | Retry the same request. If it repeats, check communication provider health. | Retry; if persistent, contact Samparka. |
| `400 Invalid purchase QR request` on purchase QR | Missing/empty `items`, non-positive `amount`, bad `currency`, missing `event_type`, invalid `event_type`, missing `payload`, invalid `created_at`, Σ(items) ≠ amount | Inspect the request body against the [Payload Reference](./payload-reference). | Correct the fields and resend. |
| `401 Integration key mismatch` on purchase QR | Body `integrationKey` doesn't match the store's integration key | Compare the `integrationKey` in the request body with the store's integration key. | Use the correct integration key for the store. |
| `403 Unsupported capability` on purchase QR | Provider does not have `submitPurchase` capability enabled | Check the provider's capabilities in the dashboard. | Enable `submitPurchase` capability for the provider. |
| `403 Store scope required` on purchase QR | API key is not bound to a store | Verify the API key is bound to a store. | Bind the API key to a store or use a different key. |
| `404 Integration not found` on purchase QR | The store bound to the API key has no integration | Config/onboarding issue. | Notify store admin. |
| `409 Purchase QR unavailable` on purchase QR | The session for this `order_id` was already completed/claimed/failed | Check the session status. | Do **not** retry. Print receipt without QR. |
| `410 Purchase QR expired` on purchase QR | The session for this `order_id` expired | Check the session expiry. | Do **not** retry. Print receipt without QR. |
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

| Problem | Likely Cause | Verification Steps | Resolution |
| ------- | ------------ | ------------------ | ---------- |
| `409 system_not_connected` on purchase QR | The System integration is not `CONNECTED`/`ACTIVE` (e.g. status `CREATED`) | Confirm the integration is connected in the Samparka dashboard and the token belongs to that integration. | Connect the System integration, then retry the `purchase-qr` call. |
| `422 No active connected communication provider is configured` on purchase QR | The System is connected but no WhatsApp/Wapio communication provider is active for the store | Confirm a communication provider is configured and active for the store. | Activate/connect the communication provider, then retry. |
| `409 This session has already been processed` on purchase QR | The same request was previously completed/claimed/failed | Check the session status. | Make a new request for a new sale; a completed session cannot get a fresh QR. |
| `410 This session's checkout has expired` on purchase QR | The same request previously expired | Check the session expiry. | Make a new request for a new sale. |
| `502 QR link could not be generated` on purchase QR | Session created but no short URL (Wapio/redirect failure) | Retry the same request. If it repeats, check communication provider health. | Retry; if persistent, contact Samparka. |
| `400 Invalid purchase QR request` on purchase QR | Missing/empty `items`, non-positive `amount`, bad `currency`, missing `event_type`, invalid `event_type`, missing `payload`, invalid `created_at`, Σ(items) ≠ amount | Inspect the request body against the [Payload Reference](./payload-reference). | Correct the fields and resend. |
| `401 Integration key mismatch` on purchase QR | Body `integrationKey` doesn't match the store's integration key | Compare the `integrationKey` in the request body with the store's integration key. | Use the correct integration key for the store. |
| `403 Unsupported capability` on purchase QR | Provider does not have `submitPurchase` capability enabled | Check the provider's capabilities in the dashboard. | Enable `submitPurchase` capability for the provider. |
| `403 Store scope required` on purchase QR | API key is not bound to a store | Verify the API key is bound to a store. | Bind the API key to a store or use a different key. |
| `404 Integration not found` on purchase QR | The store bound to the API key has no integration | Config/onboarding issue. | Notify store admin. |
| `409 Purchase QR unavailable` on purchase QR | The session for this `order_id` was already completed/claimed/failed | Check the session status. | Do **not** retry. Print receipt without QR. |
| `410 Purchase QR expired` on purchase QR | The session for this `order_id` expired | Check the session expiry. | Do **not** retry. Print receipt without QR. |

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes, idempotency, and new envelope body format.
  </Card>
  <Card title="Testing & Verification" icon="flask-conical" href="/integrations/system/purchase-qr/testing">
    Automated test cases and deployment notes.
  </Card>
</Columns>
  </Tab>
</Tabs>
