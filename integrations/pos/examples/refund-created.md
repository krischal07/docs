---
title: Refund Created Example
description: Example of a refund webhook request and successful acknowledgment.
sidebarTitle: Refund Created
---


## Request

Use the `refund_created_request` fixture from [`payloads.json`](./payloads.json).

```json
{
  "event_type": "refund.created",
  "order_id": "pos-sale-1001",
  "created_at": "2026-06-08T11:30:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": { "phone": "9800000101" },
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

Samparka accepted the refund webhook and used the sale identifier to match the original sale.

## What To Do Next

If you retry refund deliveries, resend the exact same payload once and confirm the duplicate acknowledgment.
