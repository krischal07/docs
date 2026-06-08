---
title: Payload Reference
description: Required, optional, and ignored RestroX payload fields for Samparka webhook delivery.
sidebarTitle: Payload Reference
---

# Payload Reference

Samparka reads the fields below from RestroX webhooks.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61

## Required Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` | string | Yes | RestroX event name. `type` is also accepted if `event_type` is missing. | `order.completed` |
| `order_id` or `transaction_id` or `id` | string | Yes | Sale or refund identifier used as the transaction reference. | `restrox-sale-1001` |
| `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | Yes | Outlet identifier used for outlet mapping. | `ktm-branch-01` |

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:20-27
samparka-backend/src/integrations/pos/providers/restrox/parser.js:43-48

## Optional Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `created_at` or `timestamp` | string | No | Event time from RestroX. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Customer phone number. | `9800000101` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Human-readable outlet name. | `Kathmandu Branch` |

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:29-61

## Ignored By Samparka If Present

Any fields outside the parser mappings above are not required for the canonical webhook flow.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
