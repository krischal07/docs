---
title: Idempotency
description: Root-level duplicate handling summary for POS.
sidebarTitle: Idempotency
---


<Tabs>
  <Tab title="App">
Duplicate POS deliveries return `200 Event already processed`.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR endpoint is also idempotent. Each request creates a **new checkout session** (fresh `ps_…` `purchase_reference`). Retry-safety lives in the session lifecycle:

    - `QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM` → retry returns the same QR (`200`).
    - `COMPLETED` / `CLAIMED` / failed → retry returns `409` (`already processed`).
    - Expired → retry returns `410` (`expired`).

    Make a distinct request for each new sale. The `purchase_reference` is derived from the request, so the same request for the same merchant always maps to the same session.

    See [Request / Response Contract](/integrations/pos/purchase-qr/request-response) for the full idempotency behavior.

    <Columns cols={2}>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Full idempotency behavior and session lifecycle.
      </Card>
      <Card title="Testing & Verification" icon="flask-conical" href="/integrations/pos/purchase-qr/testing">
        Automated test cases for idempotent retries.
      </Card>
    </Columns>
  </Tab>
</Tabs>
