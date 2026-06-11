---
title: Binding Mismatch
description: Example showing the restaurant binding mismatch response.
sidebarTitle: Binding Mismatch
---

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-2001",
  "created_at": "2026-06-08T13:10:00.000Z",
  "amount": 600,
  "currency": "NPR",
  "restaurantId": "99999",
  "restaurantName": "Wrong Branch"
}
```

Expected response:

```json
{
  "success": false,
  "message": "Restaurant binding mismatch"
}
```
