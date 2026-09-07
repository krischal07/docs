---
title: Authentication
description: Root-level authentication summary for System.
sidebarTitle: Authentication
---


<Tabs>
  <Tab title="App">
    - merchant lifecycle APIs use merchant auth
    - partner APIs use `Authorization: Bearer {{providerApiKey}}` as the canonical auth model
    - for this integration, use `restrox` as the `{provider}` value in documented partner and webhook routes
    - System customer APIs use `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key`
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR endpoint uses Bearer provider API key auth:

    - `Authorization: Bearer <provider_api_key>` — confirms the System vendor (provider key `slug` must match the route `:provider`). The store is resolved from the API key's `store_id` scope.

    The provider API key confirms the vendor, and the key's `store_id` scope resolves the store and integration.

    ```bash
    curl -X POST https://server.samparka.co/integrations/system/{provider}/purchase-qr \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <provider_api_key>" \
      -d '{
        "amount": 1250,
        "currency": "NPR",
        "items": [
          { "name": "Cappuccino", "qty": 1, "price": 850 }
        ]
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
