---
title: Authentication
description: Root-level authentication summary for POS.
sidebarTitle: Authentication
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
- merchant lifecycle APIs use merchant auth
- partner APIs use `Authorization: Bearer {{providerApiKey}}` as the canonical auth model
- for this integration, use `restrox` as the `{provider}` value in documented partner and webhook routes
- POS customer APIs use `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key`
- `x-partner-key` remains legacy compatibility behavior and should not be used in new integrations
- webhooks use `POST /webhook/restrox/{token}`
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR endpoint uses a two-part auth model:

    - `Authorization: Bearer <provider_api_key>` — confirms the POS vendor (provider key `slug` must match the route `:provider`)
    - `X-Integration-Key` in request body — confirms which store/outlet

    No `webhook_token` is used on this endpoint.

    ```bash
    curl -X POST https://server.samparka.xyz/integrations/pos/{provider}/purchase-qr \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <provider_api_key>" \
      -d '{
        "amount": 1250,
        "currency": "NPR",
        "X-Integration-Key": "<integration_key>",
        "items": [
          { "name": "Cappuccino", "qty": 1, "price": 850 }
        ]
      }'
    ```

    <Columns cols={2}>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Full auth details, request/response shape, and error codes.
      </Card>
      <Card title="Integration Guide" icon="rocket" href="/integrations/pos/purchase-qr/integration-guide">
        Example curl calls, behavior matrix, and design choices.
      </Card>
    </Columns>
  </Tab>
</Tabs>
