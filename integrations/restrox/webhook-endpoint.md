---
title: Webhook Endpoint
description: Configure the RestroX webhook endpoint and routing behavior.
sidebarTitle: Webhook Endpoint
---

RestroX sends webhooks to:

```text
POST /webhook/restrox/{token}
```

## Routing

For outlet-owned RestroX, the token resolves directly to `PosIntegration.webhook_token`.

The token is integration-level. It is not scoped to a separate location record for RestroX.

## Required Payload Identity

The payload should include the same restaurant identifier that was bound during partner connect:

- `restaurantId`
- or `external_location_id`
- or another supported alias in the parser

If the payload restaurant identifier does not match the stored binding, Samparka returns `409 Restaurant binding mismatch` and marks integration health as `ERROR`.
