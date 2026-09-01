---
title: Webhook Endpoint
description: Configure the POS webhook endpoint, request shape, and acknowledgment behavior.
sidebarTitle: Webhook Endpoint
---


<Tabs>
  <Tab title="App">
    ## Endpoint

    - Method: `POST`
    - Path: `/webhook/{provider}/{token}`
    - Content-Type: `application/json`
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The **Purchase QR** endpoint is the sister checkout entry point that uses the same tokenized auth as the webhook endpoint — but instead of pushing a *completed* sale, it creates a QR session and returns a QR link in the response.

    ## Endpoint

    - Method: `POST`
    - Path: `/integrations/pos/{provider}/{token}/purchase-qr`
    - Content-Type: `application/json`

    ## Purpose

    The POS calls this to get a QR link to print on the customer's receipt. The customer scans the QR → WhatsApp deep link → claim → points. It reuses the same `webhook_token` for authentication.

    <Columns cols={2}>
      <Card title="Integration Guide" icon="rocket" href="/integrations/pos/purchase-qr/integration-guide">
        Example curl calls and behavior matrix.
      </Card>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Request/response shape, error codes, idempotency.
      </Card>
    </Columns>
  </Tab>
</Tabs>
