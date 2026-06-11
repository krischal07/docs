---
title: Quick Start
description: Configure RestroX with Samparka using the outlet-owned architecture.
sidebarTitle: Quick Start
---

This is the fastest path to a working RestroX integration.

## 1. Create The Integration

```json
{
  "provider": "restrox",
  "storeId": "685000000000000000000010",
  "outletId": "685000000000000000000001"
}
```

If the store has no outlets, create an outlet before connecting RestroX.

## 2. Share The Integration Key

Creation returns an Integration Key and an integration-level webhook URL.

## 3. Connect The Restaurant

```json
{
  "integrationKey": "SPK-RX-ABC12345",
  "account": {
    "id": "restrox-account-001",
    "name": "Java Express"
  },
  "locations": [
    {
      "restaurantId": "12345",
      "restaurantName": "Kathmandu Branch"
    }
  ]
}
```

Samparka stores the restaurant binding on:

- `external_location_id`
- `external_location_name`

## 4. Send The First Sale

`POST /webhook/restrox/{token}`

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": {
    "phone": "9800000101"
  },
  "restaurantId": "12345",
  "restaurantName": "Kathmandu Branch",
  "items": [
    {
      "name": "Cappuccino",
      "qty": 1,
      "price": 850
    }
  ]
}
```

The first valid sale moves the integration from `CONNECTED` to `ACTIVE`.
