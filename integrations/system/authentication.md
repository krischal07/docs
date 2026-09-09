---
title: Authentication
description: Root-level authentication summary for System.
sidebarTitle: Authentication
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
    - merchant lifecycle APIs use merchant auth
    - partner APIs use `Authorization: Bearer {{providerApiKey}}` as the canonical auth model
    - use your assigned provider slug as the `{provider}` value in documented partner and webhook routes
    - System customer APIs use `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key`
    - `x-partner-key` remains legacy compatibility behavior and should not be used in new integrations
    - webhooks use `POST /webhook/{provider}/{token}`
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR endpoint uses a webhook-style envelope with Bearer provider API key auth:

    - `Authorization: Bearer <provider_api_key>` — confirms the System vendor (provider key `slug` must match the route `:provider`). The store is resolved from the API key's `store_id` scope.
    - `integrationKey` in the request body — the store's integration key, validated against the store resolved from the API key's scope.

    ```bash
    curl -X POST https://server.samparka.co/integrations/system/{provider}/purchase-qr \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <provider_api_key>" \
      -d '{
        "integrationKey": "SPK-RX-TTMFHBYZ",
        "payload": {
          "event_type": "order.completed",
          "order_id": "ORDER-1001",
          "amount": 1250,
          "currency": "NPR",
          "items": [
            { "name": "Cappuccino", "qty": 1, "price": 850 }
          ]
        }
      }'
    ```

    <Columns cols={2}>
      <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
        Full auth details, request/response shape, and error codes.
      </Card>
      <Card title="Integration Guide" icon="rocket" href="/integrations/system/purchase-qr/integration-guide">
        Example curl calls, behavior matrix, and design choices.
      </Card>
    </Columns>
  </Tab>
</Tabs>
