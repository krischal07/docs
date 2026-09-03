---
title: Event Types
description: Review the POS webhook events supported by Samparka Loyalty.
sidebarTitle: Event Types
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
Samparka's recommended POS event types are:

See also: [Payload Reference](./payload-reference).

- `order.completed` for completed sales
- `refund.created` for refunds
- `order.voided` for voided or canceled sales
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

| Entry point | Events | Route |
|---|---|---|
| Webhooks and Purchase QR | `order.completed`, `refund.created`, `order.voided` | `POST /integrations/pos/{provider}/purchase-qr` |

The Purchase QR endpoint creates a checkout session that flows through the same event pipeline as standard webhooks.

<Columns cols={2}>
  <Card title="Overview" icon="compass" href="/integrations/pos/purchase-qr/overview">
    Summary, architecture, and lifecycle.
  </Card>
  <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
    Request/response shape, error codes, idempotency.
  </Card>
</Columns>
  </Tab>
</Tabs>
