---
title: Payload Reference
description: Required, recommended, and optional RestroX webhook fields.
sidebarTitle: Payload Reference
---

Samparka reads the fields below from RestroX webhook payloads.

| Field | Type | Required For Transport | Notes |
| ------ | ------ | ------ | ----- |
| `event_type` or `type` | string | Yes | Missing `event_type` returns `400`. |
| `order_id` or `transaction_id` or `id` | string | No | Used for idempotency when present. |
| `restaurantId` or `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | Recommended | Must match the stored restaurant binding for outlet-owned RestroX. |
| `restaurantName` or `external_location_name` or `location_name` or `branch_name` | string | No | Stored as the human-readable restaurant name when available. |
| `amount` | number | No | Consumed by downstream event processing. |
| `currency` | string | No | Example: `NPR`. |
| `customer.phone` | string | No | Phone normalization defaults to Nepal when no country prefix is provided. |
| `created_at` or `timestamp` | string | No | Event timestamp. |

## Binding Rule

When the integration is outlet-owned, the effective restaurant identity must match `PosIntegration.external_location_id`.
