---
title: Location Binding Reference
description: Understand how the connected location binding is used during webhook delivery.
sidebarTitle: Location Binding
---

# Location Binding Reference

POS connect is outlet-owned and singular:

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
