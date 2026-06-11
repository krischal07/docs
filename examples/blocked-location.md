---
title: Blocked Location Example
description: Example showing a webhook accepted for delivery while location setup still needs review.
sidebarTitle: Blocked Location
---

# Blocked Location Example

## Request

Use the `blocked_location_request` fixture from [`payloads.json`](./payloads.json).

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-2001",
  "created_at": "2026-06-08T13:10:00.000Z",
  "amount": 600,
  "currency": "NPR",
  "customer": { "phone": "9800000103" },
  "restaurantId": "99999",
  "restaurantName": "Unknown Branch",
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

Samparka accepted the delivery, but the outlet mapping should be reviewed because the payload location did not match a processable configured location.

## What To Do Next

Confirm the correct `restaurantId` with Samparka and resend future events with the mapped location value.
