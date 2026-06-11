---
title: Location Mapping
description: Map RestroX outlet identifiers to Samparka locations before sending production traffic.
sidebarTitle: Location Mapping
---

# Location Mapping

Samparka uses `restaurantId` to match a RestroX webhook to the correct outlet configuration. The webhook token identifies the configured location record, and the payload location must also match the expected synced location value.

## What RestroX Needs To Send

Preferred fields:

- `restaurantId`
- `restaurantName`

Accepted location ID aliases:

- `external_location_id`
- `location_id`
- `outlet_id`
- `branch_id`

Accepted location name aliases:

- `external_location_name`
- `location_name`
- `branch_name`
- `outlet_name`

`restaurantId` is the permanent RestroX restaurant/location identifier and Samparka's authoritative mapping key for RestroX payloads. It is stored internally as `external_location_id`.

## Merchant Without Outlets

For merchants without outlets, `restaurantId` maps directly to the store-level location and no `outletId` is required.

## What Response To Expect

If the token is valid, Samparka can still return `200 Event received` even when the location payload does not match a valid outlet mapping. That acknowledgment means the delivery was accepted, not necessarily credited.

## What To Do If Something Goes Wrong

If a sale is accepted but expected loyalty activity is missing, verify:

1. The payload includes a location identifier.
2. The location identifier matches the configured Samparka outlet mapping.
3. The location mapping is enabled for participation.
