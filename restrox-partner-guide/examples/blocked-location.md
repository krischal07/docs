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
  "external_location_id": "unknown-branch-99",
  "external_location_name": "Unknown Branch",
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

Source:
samparka-backend/src/integrations/pos/controller.js:246-349
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77

## What To Do Next

Confirm the correct `external_location_id` with Samparka and resend future events with the mapped location value.
