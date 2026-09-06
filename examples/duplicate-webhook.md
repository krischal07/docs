---
title: Duplicate Webhook Example
description: Example showing how Samparka safely acknowledges a repeated System webhook.
sidebarTitle: Duplicate Webhook
---

import { Tabs, Tab } from "@mintlify/components";

# Duplicate Webhook Example

<Tabs>
  <Tab title="App">

## Request

Use the `duplicate_sale_request` fixture from [`payloads.json`](./payloads.json). This fixture intentionally matches the earlier sale payload.

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-1001",
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
  "message": "Event already processed"
}
```

## What Happened

Samparka recognized the repeated delivery and acknowledged it safely without handling it as a new sale.

## What To Do Next

Treat this response as success and stop retrying that payload.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

Use the `duplicate_sale_request` fixture from [`payloads.json`](./payloads.json). This fixture intentionally matches the earlier sale payload.

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-1001",
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
  "message": "Event already processed"
}
```

## What Happened

Samparka recognized the repeated delivery and acknowledged it safely without handling it as a new sale.

## What To Do Next

Treat this response as success and stop retrying that payload.
  </Tab>
</Tabs>
