---
title: Quick Start
description: Configure RestroX webhooks with Samparka Loyalty and validate the happy path quickly.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a working RestroX integration.

See also: [Testing Guide](./testing-guide) and [Integration Checklist](./integration-checklist).

## 1. Receive Outlet Mapping Details

Ask Samparka for the webhook URL and confirm the `external_location_id` value that RestroX will send for each outlet. Samparka resolves webhook tokens to a location record and also checks the `external_location_id` in the payload.

## 2. Configure the Webhook URL

Send `POST` requests to:

`/webhook/restrox/{token}`

Use `Content-Type: application/json`.

## 3. Send a Test Sale

Use the canonical sale fixture from [`examples/payloads.json`](./examples/payloads.json):

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": { "phone": "9800000101" },
  "external_location_id": "ktm-branch-01",
  "external_location_name": "Kathmandu Branch",
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## 4. Repost the Same Payload Once

Send the exact same body again. Samparka should acknowledge the duplicate safely.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## 5. Send a Test Refund

Use a `refund.created` payload that reuses the original sale identifier.

## 6. Complete Go-Live Validation

Run the checks in [Integration Checklist](./integration-checklist) before switching to production traffic.
