---
title: Response Reference
description: Reference the partner-visible response contracts for connect, test-sale, customer APIs, and webhooks.
sidebarTitle: Response Reference
---


These are the partner-visible responses for the active POS integration path.

<Info>
**Also see — Purchase QR responses** (200 QR ready, 409 pos_not_connected / already processed, 410 expired, 422, 502): [Request / Response Contract](/integrations/pos/purchase-qr/request-response).
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
    "email": "pos-sale-1001@example.com",
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
    "email": "pos-sale-1001@example.com",
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

`POST /integrations/pos/{provider}/{token}/purchase-qr`

See [Purchase QR Checkout](./purchase-qr/request-response).

### `200 Purchase QR ready`

```json
{
  "success": true,
  "message": "Purchase QR ready",
  "data": {
    "qr_link": "https://samparka.co/r/xKd93k",
    "purchase_reference": "posqr:{provider}:...",
    "amount": 1250,
    "currency": "NPR",
    "status": "QR_GENERATED",
    "expires_at": "2026-08-31T07:40:08.000Z"
  }
}
```

`qr_link` is the short URL the POS turns into a scannable QR. A retried request while the session is pending returns the same QR.

### `400 Invalid purchase QR request`

```json
{
  "success": false,
  "message": "Invalid purchase QR request",
  "errors": { "code": "purchase_qr_validation_failed" }
}
```

### `401 Invalid integration token`

```json
{
  "success": false,
  "message": "Invalid or unknown integration token"
}
```

### `404 Unknown provider`

```json
{
  "success": false,
  "message": "Unknown provider"
}
```

### `409 POS not connected`

```json
{
  "success": false,
  "message": "{provider} is not connected for this store",
  "errors": { "code": "pos_not_connected", "status": "CREATED" }
}
```

### `409 Already processed`

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

### `422 Not mapped to an outlet`

```json
{
  "success": false,
  "message": "Integration is not mapped to an outlet"
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
