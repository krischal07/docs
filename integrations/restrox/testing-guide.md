---
title: Testing Guide
description: Validate connect, sales, refunds, duplicates, tokens, and restaurant binding before go-live.
sidebarTitle: Testing Guide
---

# Testing Guide

This guide uses the shared fixtures in [`examples/payloads.json`](./examples/payloads.json).

See also: [Refunds](./refunds) and [Troubleshooting](./troubleshooting).

Source:
samparka-backend/src/integrations/pos/partners/restrox/controller.js:9-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365

## Connect Test

```bash
curl -X POST "https://your-domain/api/partners/restrox/connect" \
  -H "Content-Type: application/json" \
  -H "x-partner-key: {{partnerKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "restaurantId": "{{expectedRestaurantId}}",
    "restaurantName": "{{expectedRestaurantName}}"
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "RestroX connected",
  "connected": true,
  "integrationId": "replace-with-real-id",
  "externalLocationId": "{{expectedRestaurantId}}",
  "externalLocationName": "{{expectedRestaurantName}}",
  "status": "CONNECTED"
}
```

## Legacy Connect Rejection Test

```bash
curl -X POST "https://your-domain/api/partners/restrox/connect" \
  -H "Content-Type: application/json" \
  -H "x-partner-key: {{partnerKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "locations": [
      {
        "restaurantId": "abc"
      }
    ]
  }'
```

Expected response:

```json
{
  "success": false,
  "message": "restaurantId is required"
}
```

## Sale Test

```bash
curl -X POST "https://your-domain/webhook/restrox/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "restrox-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "external_location_id": "ktm-branch-01",
    "external_location_name": "Kathmandu Branch",
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

Use the `sale_completed_request` fixture values from [`examples/payloads.json`](./examples/payloads.json).

Source:
samparka-backend/src/integrations/pos/controller.js:351-365

## Refund Test

```bash
curl -X POST "https://your-domain/webhook/restrox/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "refund.created",
    "order_id": "restrox-sale-1001",
    "created_at": "2026-06-08T11:30:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "external_location_id": "ktm-branch-01",
    "external_location_name": "Kathmandu Branch",
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

Source:
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38
samparka-backend/src/integrations/pos/controller.js:351-365

## Void Test

```bash
curl -X POST "https://your-domain/webhook/restrox/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.voided",
    "order_id": "restrox-sale-1002",
    "created_at": "2026-06-08T12:00:00.000Z",
    "amount": 450,
    "currency": "NPR",
    "customer": { "phone": "9800000102" },
    "external_location_id": "ktm-branch-01",
    "external_location_name": "Kathmandu Branch",
    "items": [{ "name": "Americano", "qty": 1, "price": 450 }]
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:23-26
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38
samparka-backend/src/integrations/pos/controller.js:351-365

## Duplicate Webhook Test

Resend the sale test payload exactly as-is.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

Source:
samparka-backend/src/integrations/pos/controller.js:293-312

## Invalid Token Test

```bash
curl -X POST "https://your-domain/webhook/restrox/{invalid-token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "restrox-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "external_location_id": "ktm-branch-01",
    "external_location_name": "Kathmandu Branch",
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

Source:
samparka-backend/src/integrations/pos/controller.js:200-205

## Wrong Location Test

```bash
curl -X POST "https://your-domain/webhook/restrox/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "restrox-sale-2001",
    "created_at": "2026-06-08T13:10:00.000Z",
    "amount": 600,
    "currency": "NPR",
    "customer": { "phone": "9800000103" },
    "external_location_id": "unknown-branch-99",
    "external_location_name": "Unknown Branch",
    "items": [{ "name": "Latte", "qty": 1, "price": 600 }]
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

What this means: Samparka accepted the delivery, but location setup should be verified before go-live.
What this means: Samparka accepted the delivery, but the webhook restaurant identifier does not match the connected binding.

Source:
samparka-backend/src/integrations/pos/controller.js:246-349
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77
