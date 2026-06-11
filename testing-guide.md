---
title: Testing Guide
description: Validate sales, refunds, duplicates, tokens, and location handling before go-live.
sidebarTitle: Testing Guide
---

# Testing Guide

This guide uses the shared fixtures in [`examples/payloads.json`](./examples/payloads.json).

See also: [Refunds](./refunds) and [Troubleshooting](./troubleshooting).

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
    "restaurantId": "12345",
    "restaurantName": "Kathmandu Branch",
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
    "restaurantId": "12345",
    "restaurantName": "Kathmandu Branch",
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
    "restaurantId": "12345",
    "restaurantName": "Kathmandu Branch",
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
curl -X POST "https://your-domain/webhook/restrox/{invalid-token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "restrox-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "restaurantId": "12345",
    "restaurantName": "Kathmandu Branch",
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
    "restaurantId": "99999",
    "restaurantName": "Unknown Branch",
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
