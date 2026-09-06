---
title: Endpoint Catalog
description: Root-level endpoint summary for System.
sidebarTitle: Endpoint Catalog
---


<Tabs>
  <Tab title="App">
| Endpoint | Purpose | Authentication |
| -------- | ------- | -------------- |
| `POST /api/partners/{provider}/connect` | Bind one external location to one outlet-owned Samparka integration. | `Authorization: Bearer {{providerApiKey}}` |
| `POST /api/partners/{provider}/test-sale` | Submit a partner-side test sale into the webhook processing path. | `Authorization: Bearer {{providerApiKey}}` |
| `GET /api/partners/{provider}/customers/search` | Search for one customer by phone within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
| `GET /api/partners/{provider}/customers/{customerId}` | Fetch one customer record within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    | Endpoint | Purpose | Authentication |
    | -------- | ------- | -------------- |
    | `POST /integrations/system/{provider}/purchase-qr` | Create a scannable purchase-checkout QR for a cash sale from the System terminal; returns the QR link in the response. | `Authorization: Bearer {{providerApiKey}}` and `X-Integration-Key` in request body |

    <Columns cols={2}>
      <Card title="Overview" icon="compass" href="/integrations/system/purchase-qr/overview">
        Summary, architecture, and lifecycle.
      </Card>
      <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
        Request/response shape, error codes, idempotency.
      </Card>
      <Card title="Integration Guide" icon="rocket" href="/integrations/system/purchase-qr/integration-guide">
        Example curl and behavior matrix.
      </Card>
      <Card title="Testing & Verification" icon="flask-conical" href="/integrations/system/purchase-qr/testing">
        Automated test cases and deployment notes.
      </Card>
    </Columns>
  </Tab>
</Tabs>
