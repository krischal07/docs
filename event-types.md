---
title: Event Types
description: Root-level event type summary for POS.
sidebarTitle: Event Types
---


<Tabs>
  <Tab title="App">
- `sale.completed`
- `refund.created`
- `sale.voided`
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR endpoint does not use webhook event types. Instead, the POS sends a `POST` request with `amount`, `currency`, and `items` to create a checkout session. The customer then completes the loyalty flow by scanning the QR on their receipt.

    | Concern | App (webhook events) | Communication (Purchase QR) |
    | ------- | -------------------- | --------------------------- |
    | Entry point | `order.completed`, `refund.created`, `order.voided` | `POST /integrations/pos/{provider}/purchase-qr` |
    | Payload | Event type + sale details | `amount` + `currency` + `items` |
    | Customer resolution | Phone in webhook payload | On scan & claim |
    | Points awarded | When webhook is processed | After WhatsApp verification |

    <Columns cols={2}>
      <Card title="Overview" icon="compass" href="/integrations/pos/purchase-qr/overview">
        Summary, architecture, state lifecycle, and end-to-end sequence.
      </Card>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Request/response shape, error codes, and idempotency.
      </Card>
    </Columns>
  </Tab>
</Tabs>
