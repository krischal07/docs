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

Samparka accepted the sale webhook, resolved the integration from the webhook token, and continued processing it as a completed sale event.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-22
samparka-backend/src/integrations/pos/controller.js:351-365

## What To Do Next

Repeat the same payload once to confirm duplicate handling, then continue with the refund test.
