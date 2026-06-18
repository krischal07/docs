---
title: Missing Binding Example
description: Example showing the response returned when an outlet-owned POS integration has no bound restaurant.
sidebarTitle: Missing Binding
---


## Request

Use the `missing_binding_request` fixture from [`payloads.json`](./payloads.json) against the token for a newly created integration before connect.

```json
{
  "event_type": "order.completed",
  "order_id": "pos-sale-2001",
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
  "success": false,
  "message": "Integration is not connected to a restaurant"
}
```

## What Happened

Samparka resolved the integration from the webhook token, then rejected the event because no restaurant was connected yet.

## What To Do Next

Connect the restaurant first, confirm the binding is stored on the integration, and then resend future events.
