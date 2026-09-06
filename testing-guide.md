---
title: Testing Guide
description: Validate connect, test-sale, customer verification, webhook delivery, refunds, duplicates, and failure paths before go-live.
sidebarTitle: Testing Guide
---

import { Tabs, Tab } from "@mintlify/components";

# Testing Guide

<Tabs>
  <Tab title="App">

This guide uses the shared fixtures in [`examples/payloads.json`](./examples/payloads.json).

See also: [Refunds](./refunds) and [Troubleshooting](./troubleshooting).

## Integration Verification Flow

```txt
Create Integration
↓
Connect Location
↓
Store Returned Token
↓
Send Sale Or Submit Test Sale
↓
Verify Integration Became ACTIVE
↓
Verify Customer Exists
↓
Verify Loyalty Transaction Exists
↓
Verify Points Awarded
```

## Connect Test

```bash
curl -X POST "https://your-domain/api/partners/{provider}/connect" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "externalLocationId": "{{expectedLocationId}}",
    "externalLocationName": "{{expectedLocationName}}"
  }'
```

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName`, but new integrations should send `externalLocationId` and `externalLocationName`.

Expected success response:

```json
{
  "success": true,
  "integrationId": "replace-with-real-id",
  "token": "replace-with-real-token",
  "status": "CONNECTED"
}
```

Validate that the response:

- returns HTTP `200`
- has `"success": true`
- has `"status": "CONNECTED"`
- includes a non-empty `"integrationId"`
- includes a non-empty `"token"`

Do not assert `message`, `restaurantId`, or `externalLocationId` in the connect response. Those values are no longer part of the success payload.

## Test Sale Wrapper Check

```bash
curl -X POST "https://your-domain/api/partners/{provider}/test-sale" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "payload": {
      "event_type": "order.completed",
      "order_id": "system-sale-1001",
      "created_at": "2026-06-08T10:15:00.000Z",
      "amount": 850,
      "currency": "NPR",
      "customer": {
        "phone": "9800000101"
      },
      "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
    }
  }'
```

Expected response:

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

Duplicate wrapper response:

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

## Direct Webhook Sale Test

```bash
curl -X POST "https://your-domain/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": {
      "phone": "9800000101"
    },
    "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## Verify ACTIVE

After the first valid sale, fetch the merchant integration and confirm:

- `connectionStatus = ACTIVE`
- `healthStatus = HEALTHY`

`ACTIVE` only proves the integration activated. It does not prove business processing succeeded.

## Customer Search After First Sale

```bash
curl -X GET "https://your-domain/api/partners/{provider}/customers/search?phone=9800000101" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected hit response:

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

Expected miss response:

```json
{
  "exists": false
}
```

## Customer Detail Verification

```bash
curl -X GET "https://your-domain/api/partners/{provider}/customers/{customerId}" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected response:

```json
{
  "customer": {
    "id": "684a00000000000000001001",
    "phone": "9800000101",
    "points": 85,
    "lifetimePoints": 85
  }
}
```

## Loyalty Transaction Verification

After customer lookup succeeds, confirm the test sale created a loyalty transaction in merchant tooling or by verifying that the customer's points changed as expected.

## Refund Test

```bash
curl -X POST "https://your-domain/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "refund.created",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T11:30:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": {
      "phone": "9800000101"
    },
    "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## Duplicate Webhook Test

Resend the sale test payload exactly as-is.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## Invalid Token Test

```bash
curl -X POST "https://your-domain/webhook/{provider}/{invalid-token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
  }'
```

Expected response:

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

## Missing Location Binding Test

Use the token from a newly created integration before calling connect:

```bash
curl -X POST "https://your-domain/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "system-sale-2001",
    "amount": 600,
    "currency": "NPR",
    "customer": { "phone": "9800000103" }
  }'
```

Expected response:

```json
{
  "success": false,
  "message": "Integration is not connected to a restaurant"
}
```

## Disconnected Integration Test

After disconnecting the integration, submit the same webhook again.

Expected response:

```json
{
  "success": false,
  "message": "Integration disconnected"
}
```

## Validation Failure Tests

Non-object body response:

```json
{
  "success": false,
  "message": "Request body must be a JSON object"
}
```

Missing event type response:

```json
{
  "success": false,
  "message": "Missing event_type in payload"
}
```
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

This guide covers Purchase QR testing. See [Testing & Verification](./integrations/system/purchase-qr/testing) for the automated test suite.

## Prerequisites

Before testing Purchase QR, confirm:

- the System integration is `CONNECTED`/`ACTIVE`
- an active WhatsApp/Wapio communication provider is configured for the store

## Happy Path

```bash
curl -X POST "https://your-domain/integrations/system/{provider}/purchase-qr" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <provider_api_key>" \
  --data '{
    "amount": 1250,
    "currency": "NPR",
    "X-Integration-Key": "<integration_key>",
    "items": [
      { "name": "Cappuccino", "qty": 1, "price": 850 }
    ]
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "Purchase QR ready",
  "data": {
    "qr_link": "https://your-domain/r/xKd93k",
    "purchase_reference": "ps_1690000000000_ab12cd34ef56",
    "amount": 1250,
    "currency": "NPR",
    "status": "QR_GENERATED",
    "expires_at": "2026-08-31T07:40:08.000Z"
  }
}
```

Validate that:

- HTTP `200`
- `"success": true`
- `"status": "QR_GENERATED"`
- `qr_link` is a non-empty scannable short URL

## Idempotent Retry

Each request creates a **new checkout session** (fresh `ps_…` reference). Retry-safety lives in the session lifecycle: re-sending the request while the session is pending (`QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM`) returns the same `qr_link` (200), not a new session.

## Validation Failure

Send a body with a non-positive or missing `amount`, or missing/empty `items`.

Expected response:

```json
{
  "success": false,
  "message": "Invalid purchase QR request",
  "errors": { "code": "purchase_qr_validation_failed" }
}
```

## System Not Connected

Use an integration key whose System integration status is `CREATED` (not `CONNECTED`/`ACTIVE`).

Expected response:

```json
{
  "success": false,
  "message": "{provider} is not connected for this store",
  "errors": { "code": "system_not_connected", "status": "CREATED" }
}
```

## No Active Communication Provider

Use a connected System integration without an active WhatsApp/Wapio communication provider.

Expected response:

```json
{
  "success": false,
  "message": "No active connected communication provider is configured"
}
```

## Invalid Token Or Unknown Provider

Expected `401` for an unknown/invalid token and `404` for a provider not in `{blanxer, restrox}`.

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
