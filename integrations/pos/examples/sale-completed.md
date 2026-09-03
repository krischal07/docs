---
title: Sale Completed Example
description: Example of a successful completed-sale webhook request and acknowledgment.
sidebarTitle: Sale Completed
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
## Request

Use the `sale_completed_request` fixture from [`payloads.json`](./payloads.json).

```json
{
  "event_type": "order.completed",
  "order_id": "pos-sale-1001",
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

## What To Do Next

Verify the integration becomes `ACTIVE`, then repeat the same payload once to confirm duplicate handling.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

## Request

Use the `sale_completed_request` fixture from [`payloads.json`](./payloads.json).

```json
{
  "event_type": "order.completed",
  "order_id": "pos-sale-1001",
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

## What To Do Next

Verify the integration becomes `ACTIVE`, then repeat the same payload once to confirm duplicate handling.

```bash
curl -X POST https://server.samparka.xyz/integrations/pos/{provider}/purchase-qr \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <provider_api_key>" \
  -d '{
    "amount": 1250,
    "currency": "NPR",
    "X-Integration-Key": "<integration_key>",
    "items": [
      { "name": "Cappuccino", "qty": 1, "price": 850 }
    ]
  }'
```

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
</Columns>
  </Tab>
</Tabs>
