---
title: Restaurant Binding Reference
description: Understand how the connected restaurant binding is used during webhook delivery.
sidebarTitle: Restaurant Binding
---


RestroX connect is outlet-owned and singular:

```txt
One Integration
=
One Outlet
=
One Restaurant
```

The `connect` step persists:

- `external_location_id`
- `external_location_name`

Webhook delivery then resolves the bound restaurant from the integration that owns the webhook token.

