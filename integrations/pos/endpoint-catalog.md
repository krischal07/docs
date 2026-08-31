---
title: Endpoint Catalog
description: Root-level endpoint summary for POS.
sidebarTitle: Endpoint Catalog
---


<Tabs>
  <Tab title="App">
    | Endpoint | Purpose | Authentication |
    | -------- | ------- | -------------- |
    | `POST /api/partners/{provider}/connect` | Bind one external location to one outlet-owned Samparka integration. | `Authorization: Bearer {{providerApiKey}}` |
    | `POST /api/partners/{provider}/test-sale` | Submit a partner-side test sale into the webhook processing path. | `Authorization: Bearer {{providerApiKey}}` |
    | `POST /webhook/{provider}/{token}` | Deliver sale, refund, or void events to the tokenized webhook endpoint. | Tokenized URL path |
    | `GET /api/partners/{provider}/customers/search` | Search for one customer by items within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
    | `GET /api/partners/{provider}/customers/{customerId}` | Fetch one customer record within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
  </Tab>
  <Tab title="Communication">
    | Endpoint | Purpose | Authentication |
    | -------- | ------- | -------------- |
    | `POST /integrations/pos/{provider}/{token}/purchase-qr` | Create a scannable purchase-checkout QR for a cash sale from the POS terminal; returns the QR link in the response. | Tokenized URL path (same `webhook_token`) |

    <Columns cols={2}>
      <Card title="Overview" icon="compass" href="/integrations/pos/purchase-qr/overview">
        Summary, architecture, and lifecycle.
      </Card>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Request/response shape, error codes, idempotency.
      </Card>
      <Card title="Integration Guide" icon="rocket" href="/integrations/pos/purchase-qr/integration-guide">
        Example curl and behavior matrix.
      </Card>
      <Card title="Testing & Verification" icon="flask-conical" href="/integrations/pos/purchase-qr/testing">
        Automated test cases and deployment notes.
      </Card>
    </Columns>
  </Tab>
</Tabs>
