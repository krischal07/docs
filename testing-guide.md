---
title: Testing Guide
description: Validate connect, test-sale, customer verification, webhook delivery, refunds, duplicates, and failure paths before go-live.
sidebarTitle: Testing Guide
---

# Testing Guide

This guide uses the shared fixtures in [`examples/payloads.json`](./examples/payloads.json).

See also: [Refunds](./refunds) and [Troubleshooting](./troubleshooting).

## Integration Verification Flow

```txt
Create Integration
↓
Connect Restaurant
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
  -H "x-partner-key: {{partnerKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "restaurantId": "{{expectedRestaurantId}}",
    "restaurantName": "{{expectedRestaurantName}}"
  }'
```

Expected success response:

```json
{
  "success": true,
  "integrationId": "replace-with-real-id",
  "token": "replace-with-real-token",
  "status": "CONNECTED"
}
```

## Test Sale Wrapper Check

```bash
curl -X POST "https://your-domain/api/partners/{provider}/test-sale" \
  -H "Content-Type: application/json" \
  -H "x-partner-key: {{partnerKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "payload": {
      "event_type": "order.completed",
      "order_id": "pos-sale-1001",
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
    "order_id": "pos-sale-1001",
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
  -H "x-partner-key: {{partnerKey}}" \
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
    "email": "pos-sale-1001@example.com",
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
  -H "x-partner-key: {{partnerKey}}" \
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
    "order_id": "pos-sale-1001",
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
    "order_id": "pos-sale-1001",
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

## Missing Restaurant Binding Test

Use the token from a newly created integration before calling connect:

```bash
curl -X POST "https://your-domain/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "pos-sale-2001",
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
