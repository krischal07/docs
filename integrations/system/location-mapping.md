---
title: Location Binding Reference
description: Understand how the connected location binding is used during webhook delivery.
sidebarTitle: Location Binding
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
# Location Binding Reference

System connect is outlet-owned and singular:

```txt
One Integration
=
One Outlet
-> One External Location
```

The `connect` step persists:

- `external_location_id`
- `external_location_name`

Webhook delivery then resolves the bound location from the integration that owns the webhook token.

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` on connect and normalizes them to the location fields above. New integrations should send `externalLocationId` and `externalLocationName`.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

# Location Binding Reference

System connect is outlet-owned and singular:

```txt
One Integration
=
One Outlet
-> One External Location
```

The `connect` step persists:

- `external_location_id`
- `external_location_name`

Webhook delivery then resolves the bound location from the integration that owns the webhook token.

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` on connect and normalizes them to the location fields above. New integrations should send `externalLocationId` and `externalLocationName`.

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
  <Card title="Integration Guide" icon="rocket" href="/integrations/system/purchase-qr/integration-guide">
    Example curl calls and behavior matrix.
  </Card>
</Columns>
  </Tab>
</Tabs>
