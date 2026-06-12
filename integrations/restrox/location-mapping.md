---
title: Location Mapping
description: Understand how the connected restaurant binding is used during webhook delivery.
sidebarTitle: Location Mapping
---

# Location Mapping

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

Webhook delivery must then send the same restaurant identifier so the inbound event matches the connected outlet.

Source:
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/controller.js:285-365

## Connect Source Of Truth

```txt
integrationKey
-> restaurantId
-> external_location_id
```

There is no location sync, location review, location selection, or location approval step in the active connect path.

## What Webhook Delivery Needs To Send

Send one of these location fields:

- `external_location_id`
- `location_id`
- `outlet_id`
- `branch_id`

`external_location_name`, `location_name`, `branch_name`, or `outlet_name` can also be sent as an optional label.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:43-54

## What Response To Expect

If the token is valid, Samparka can still return `200 Event received` even when the location payload does not match a valid outlet mapping. That acknowledgment means the delivery was accepted, not necessarily credited.

Source:
samparka-backend/src/integrations/pos/controller.js:246-349
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77

## What To Do If Something Goes Wrong

If a sale is accepted but expected loyalty activity is missing, verify:

1. The payload includes a location identifier.
2. The location identifier matches the already connected restaurant binding.
3. The integration was connected before the webhook was sent.

Source:
samparka-backend/src/integrations/pos/controller.js:285-365
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77
