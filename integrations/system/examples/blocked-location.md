---
title: Missing Binding Example
description: Example showing the response returned when an outlet-owned System integration has no bound restaurant.
sidebarTitle: Missing Binding
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
## Request

Use the `missing_binding_request` fixture from [`payloads.json`](./payloads.json) against the token for a newly created integration before connect.

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-2001",
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
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

If the System integration is not `CONNECTED`/`ACTIVE`, the Purchase QR endpoint returns `409 system_not_connected`.

```json
{
  "success": false,
  "message": "{provider} is not connected for this store",
  "errors": { "code": "system_not_connected", "status": "CREATED" }
}
```

Connect the System integration first, then retry the `purchase-qr` call.

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
</Columns>
  </Tab>
</Tabs>
