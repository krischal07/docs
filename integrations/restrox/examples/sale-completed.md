---
title: Sale Completed
description: Example of a valid outlet-owned RestroX sale webhook.
sidebarTitle: Sale Completed
---

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "restaurantId": "12345",
  "restaurantName": "Kathmandu Branch"
}
```

Expected webhook response:

```json
{
  "success": true,
  "message": "Event received"
}
```
