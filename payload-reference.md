---
title: Payload Reference
description: Required, optional, and ignored RestroX payload fields for Samparka webhook delivery.
sidebarTitle: Payload Reference
---

# Payload Reference

Samparka reads the fields below from RestroX webhooks.

## Required Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` | string | Yes | RestroX event name. `type` is also accepted if `event_type` is missing. | `order.completed` |
| `order_id` or `transaction_id` or `id` | string | Yes | Sale or refund identifier used as the transaction reference. | `restrox-sale-1001` |
| `restaurantId` or `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | Yes | Permanent RestroX restaurant/location identifier used for outlet mapping. It is unique, stable, never changes, and is the primary mapping key. Internally stored as `external_location_id`. | `12345` |

## Optional Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `created_at` or `timestamp` | string | No | Event time from RestroX. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Customer phone number. | `9800000101` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `restaurantName` or `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Human-readable restaurant/location name. Internally stored as `external_location_name`. | `Kathmandu Branch` |

## Backward Compatibility

Preferred fields:

- `restaurantId`
- `restaurantName`

Samparka also accepts the following aliases for compatibility:

Location ID aliases:

- `externalLocationId`
- `external_location_id`
- `locationId`
- `branchId`
- `id`

Location name aliases:

- `externalLocationName`
- `external_location_name`
- `locationName`
- `branchName`
- `name`

These aliases remain supported but new integrations should use:

- `restaurantId`
- `restaurantName`

## Ignored By Samparka If Present

Any fields outside the parser mappings above are not required for the canonical webhook flow.
