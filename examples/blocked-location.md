---
title: Missing Binding Example
description: Example showing a webhook accepted for delivery while the integration still needs a restaurant binding.
sidebarTitle: Missing Binding
---

# Missing Binding Example

## Request

Use the `missing_binding_request` fixture from [`payloads.json`](./payloads.json).

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-2001",
  "created_at": "2026-06-08T13:10:00.000Z",
  "amount": 600,
  "currency": "NPR",
  "customer": { "phone": "9800000103" },
  "items": [{ "name": "Latte", "qty": 1, "price": 600 }]
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

Samparka accepted the delivery, but the integration still needs a bound restaurant before the event can be attributed correctly.

Source:
samparka-backend/src/integrations/pos/controller.js:246-349
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77

## What To Do Next

Connect the restaurant first, confirm the binding is stored on the integration, and then resend future events.
