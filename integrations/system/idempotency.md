---
title: Idempotency
description: Understand duplicate webhook handling and replay safety for System events.
sidebarTitle: Idempotency
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
Samparka accepts duplicate webhook deliveries safely. If System sends the same webhook more than once, Samparka can return `200 Event already processed` instead of creating duplicate downstream activity.

## Purchase QR Idempotency

The purchase checkout endpoint is also idempotent. See [Request / Response Contract](./purchase-qr/request-response).

The idempotency key (`purchaseReference`) is derived from the request. The same request for the same merchant always maps to the same session, so a retried request reuses the pending session's QR instead of creating a duplicate.

- `QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM` → retry returns the same QR (`200`).
- `COMPLETED` / `CLAIMED` / failed → retry returns `409` (`already processed`).
- Expired → retry returns `410` (`expired`).

Make a distinct request for each new sale.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

Samparka accepts duplicate webhook deliveries safely. If System sends the same webhook more than once, Samparka can return `200 Event already processed` instead of creating duplicate downstream activity.

## Purchase QR Idempotency

The purchase checkout endpoint is also idempotent. See [Request / Response Contract](./purchase-qr/request-response) for the full idempotency behavior.

The idempotency key (`purchaseReference`) is derived from the request. The same request for the same merchant always maps to the same session, so a retried request reuses the pending session's QR instead of creating a duplicate.

- `QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM` → retry returns the same QR (`200`).
- `COMPLETED` / `CLAIMED` / failed → retry returns `409` (`already processed`).
- Expired → retry returns `410` (`expired`).

Make a distinct request for each new sale.

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
  <Card title="Testing & Verification" icon="flask-conical" href="/integrations/system/purchase-qr/testing">
    Automated test cases and deployment notes.
  </Card>
</Columns>
  </Tab>
</Tabs>
