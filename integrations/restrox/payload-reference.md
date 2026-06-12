---
title: Payload Reference
description: Required and optional RestroX fields for both connect and webhook delivery.
sidebarTitle: Payload Reference
---

# Payload Reference

Samparka reads different fields for `connect` and for webhook delivery.

Source:
samparka-backend/src/integrations/pos/partners/restrox/controller.js:9-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61

## Connect Request

`POST /api/partners/restrox/connect`

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `PosIntegration`. | `restrox-key-001` |
| `restaurantId` | string | Yes | RestroX restaurant identifier persisted as `external_location_id`. | `ktm-branch-01` |

### Optional Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `restaurantName` | string | No | Human-readable restaurant name persisted as `external_location_name`. | `Kathmandu Branch` |

### Not Supported

The connect contract does not accept:

- `account`
- `locations`
- `locationList`
- `restaurantList`
- `branches`

Source:
samparka-backend/src/integrations/pos/partners/restrox/controller.js:9-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260

## Webhook Fields

`POST /webhook/restrox/{token}`

### Required Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` | string | Yes | RestroX event name. `type` is also accepted if `event_type` is missing. | `order.completed` |
| `order_id` or `transaction_id` or `id` | string | Yes | Sale or refund identifier used as the transaction reference. | `restrox-sale-1001` |
| `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | Yes | Restaurant identifier that must match the connected integration binding. | `ktm-branch-01` |

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:20-27
samparka-backend/src/integrations/pos/providers/restrox/parser.js:43-48

### Optional Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `created_at` or `timestamp` | string | No | Event time from RestroX. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Customer phone number. | `9800000101` |
| `customer.email` or `email` | string | No | Customer email used for downstream customer verification when available. | `restrox-sale-1001@example.com` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Human-readable outlet name. | `Kathmandu Branch` |

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:29-61

## Ignored By Samparka If Present

Any fields outside the parser mappings above are not required for the canonical webhook flow.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
