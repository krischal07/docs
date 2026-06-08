---
title: Sale Completed Example
description: Example of a successful completed-sale webhook request and acknowledgment.
sidebarTitle: Sale Completed
---

# Sale Completed Example

## Request

Use the `sale_completed_request` fixture from [`payloads.json`](./payloads.json).

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

## Response

```json
{
  "success": true,
  "message": "Event received"
}
```

## What Happened

Samparka accepted the sale webhook and continued processing it as a completed sale event.

## What To Do Next

Repeat the same payload once to confirm duplicate handling, then continue with the refund test.
