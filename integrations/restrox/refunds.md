---
title: Refunds
description: Send RestroX refund events through the outlet-owned webhook contract.
sidebarTitle: Refunds
---

Refunds use the same webhook route and restaurant binding as sales.

## Requirements

- send the same restaurant identifier that is bound to the outlet
- include a stable order or transaction identifier when possible
- preserve the same payload shape on retries

## Example

```json
{
  "event_type": "refund.created",
  "order_id": "restrox-sale-1001",
  "created_at": "2026-06-08T11:30:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "restaurantId": "12345",
  "restaurantName": "Kathmandu Branch"
}
```
