---
title: Response Reference
description: Reference the partner-visible response contracts for connect, test-sale, customer APIs, and webhooks.
sidebarTitle: Response Reference
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">

These are the partner-visible responses for the active System integration path.

<Info>
**Also see — Purchase QR responses** (200 QR ready, 409 system_not_connected / already processed, 410 expired, 422, 502): [Request / Response Contract](/integrations/system/purchase-qr/request-response).
</Info>

See also: [Troubleshooting](./troubleshooting).

## Connect Success

```json
{
  "success": true,
  "integrationId": "684915c401ec340433d84ee2",
  "token": "abcxyz",
  "status": "CONNECTED"
}
```

## Test Sale Success

```json
{
  "success": true,
  "message": "Test sale submitted",
  "data": {
    "success": true,
    "message": "Event received"
  }
}
```

## Test Sale Duplicate

```json
{
  "success": true,
  "message": "Test sale submitted",
  "data": {
    "success": true,
    "message": "Event already processed"
  }
}
```

## Customer Search Hit

```json
{
  "exists": true,
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "system-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

## Customer Search Miss

```json
{
  "exists": false
}
```

## Customer Detail

```json
{
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "system-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

## `200 Event received`

```json
{
  "success": true,
  "message": "Event received"
}
```

## `200 Event already processed`

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## `400 Request body must be a JSON object`

```json
{
  "success": false,
  "message": "Request body must be a JSON object"
}
```

## `400 Missing event_type in payload`

```json
{
  "success": false,
  "message": "Missing event_type in payload"
}
```

## `404 Invalid webhook token`

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

## `409 Integration disconnected`

```json
{
  "success": false,
  "message": "Integration disconnected"
}
```

## `409 Integration is not connected to a restaurant`

```json
{
  "success": false,
  "message": "Integration is not connected to a restaurant"
}
```

## `409 Integration is not ready to receive webhooks`

```json
{
  "success": false,
  "message": "Integration is not ready to receive webhooks"
}
```

## Partner Customer Error Envelope

```json
{
  "error": "partner_customer_error",
  "message": "Phone is required"
}
```

The same `{ error, message }` envelope is used for partner customer validation, auth, and lookup failures.

## Purchase QR

`POST /integrations/system/{provider}/purchase-qr`

Auth is the provider API key (`Authorization: Bearer <provider_api_key>`); the store is resolved from the API key's `store_id` scope. The `integration_key` is returned in the response body. See [Purchase QR Checkout](./purchase-qr/request-response).

### `200 Purchase QR ready`

```json
{
  "success": true,
  "message": "Purchase QR ready",
  "data": {
    "qr_link": "https://samparka.co/r/xKd93k",
    "purchase_reference": "sysqr:restrox:6a9e80a4...:ORDER-1001",
    "amount": 1250,
    "currency": "NPR",
    "status": "QR_GENERATED",
    "expires_at": "2026-08-26T12:12:04.000Z",
    "integration_key": "SPK-RX-TTMFHBYZ"
  }
}
```

`data.qr_link` is the short URL the POS renders as a QR. `data.integration_key` echoes the store's integration key sent in the request body. With `payload.order_id`, retried requests return the same QR and `purchase_reference`. Without `order_id`, each request creates a new session.

Sessions expire quickly (default 120 seconds or the store's configured QR expiry) — print immediately.

### `400 Invalid purchase QR request`

```json
{
  "success": false,
  "message": "Invalid purchase QR request",
  "errors": { "code": "purchase_qr_validation_failed" }
}
```

### `401 Invalid provider key or integration key mismatch`

```json
{
  "success": false,
  "message": "Invalid provider key or integration key mismatch"
}
```

### `403 Store scope required`

```json
{
  "success": false,
  "message": "API key must be bound to a store"
}
```

### `403 Unsupported capability`

```json
{
  "success": false,
  "message": "Provider does not have submitPurchase capability enabled"
}
```

### `404 Unknown provider`

```json
{
  "success": false,
  "message": "Unknown provider"
}
```

### `404 Integration not found`

```json
{
  "success": false,
  "message": "Integration not found for store"
}
```

### `409 System not connected`

```json
{
  "success": false,
  "message": "{provider} is not connected for this store",
  "errors": { "code": "system_not_connected", "status": "CREATED" }
}
```

### `409 Purchase QR unavailable`

```json
{
  "success": false,
  "message": "This session has already been processed",
  "errors": { "code": "purchase_qr_unavailable", "status": "COMPLETED" }
}
```

### `410 Expired`

```json
{
  "success": false,
  "message": "This session's checkout has expired",
  "errors": { "code": "purchase_qr_expired" }
}
```

### `422 No active connected communication provider`

```json
{
  "success": false,
  "message": "No active connected communication provider is configured"
}
```

### `502 QR generation failed`

```json
{
  "success": false,
  "message": "QR link could not be generated for this bill",
  "errors": { "code": "purchase_qr_generation_failed" }
}
```
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

`POST /integrations/system/{provider}/purchase-qr`

Auth is the provider API key (`Authorization: Bearer <provider_api_key>`); the store is resolved from the API key's `store_id` scope. The `integration_key` is returned in the response body. See [Purchase QR Checkout](/integrations/system/purchase-qr/request-response).

## 200 — Purchase QR ready

```json
{
  "success": true,
  "message": "Purchase QR ready",
  "data": {
    "qr_link": "https://samparka.co/r/xKd93k",
    "purchase_reference": "sysqr:restrox:6a9e80a4...:ORDER-1001",
    "amount": 1250,
    "currency": "NPR",
    "status": "QR_GENERATED",
    "expires_at": "2026-08-26T12:12:04.000Z",
    "integration_key": "SPK-RX-TTMFHBYZ"
  }
}
```

`data.qr_link` is the short URL the POS renders as a QR. `data.integration_key` echoes the store's integration key sent in the request body. With `payload.order_id`, retried requests return the same QR and `purchase_reference`. Without `order_id`, each request creates a new session.

Sessions expire quickly (default 120 seconds or the store's configured QR expiry) — print immediately.

## Error codes

| Status | Meaning | POS behavior |
|---|---|---|
| `400 purchase_qr_validation_failed` | Bad body — `payload` missing, `event_type` not `order.completed`, or amount/items/currency rules violated | Log it; this is a POS bug. Print receipt without QR. |
| `401` | Invalid/missing provider API key, or missing/mismatched `integrationKey` in the body | Surface config error. Print receipt without QR. |
| `403 store_scope_required` | The provider API key is not bound to a store | Contact Samparka. |
| `403 unsupported_capability` | Provider key does not have purchase submission enabled | Contact Samparka. Print receipt without QR. |
| `404 integration_not_found` | The store bound to the API key has no integration | Config/onboarding issue. Print receipt without QR. |
| `404 unknown provider` | `{provider}` slug wrong, or the API key was issued for a different provider | Config problem. |
| `409 system_not_connected` | Store integration is not `CONNECTED`/`ACTIVE` (or its key was revoked) | Config/onboarding issue. Print receipt without QR. |
| `409 purchase_qr_unavailable` | The session for this `order_id` was already completed/claimed/failed | Do **not** retry. Print receipt without QR. |
| `410 purchase_qr_expired` | The session for this `order_id` expired | Do **not** retry. Print receipt without QR. |
| `422 provider_unconfigured` | Store has no active WhatsApp/Wapio communication provider | Store-side setup issue. Print receipt without QR. |
| `502 purchase_qr_generation_failed` | Samparka-side QR/redirect failure | Safe to retry once. If it fails again, print without QR. |

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes, idempotency, and new envelope body format.
  </Card>
  <Card title="Testing & Verification" icon="flask-conical" href="/integrations/system/purchase-qr/testing">
    Automated test cases and deployment notes.
  </Card>
</Columns>

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
  <Card title="Testing & Verification" icon="flask-conical" href="/integrations/system/purchase-qr/testing">
    Automated test cases and deployment notes.
  </Card>
</Columns>
  </Tab>
</Tabs>
