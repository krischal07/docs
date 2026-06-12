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

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:20-27

### Optional Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `created_at` or `timestamp` | string | No | Event time from RestroX. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Primary customer identifier for search and loyalty attribution. | `9800000101` |
| `customer.email` or `email` | string | No | Optional customer metadata when available. | `restrox-sale-1001@example.com` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | No | Optional non-canonical restaurant metadata. Outlet-owned attribution is resolved from the integration binding instead. | `ktm-branch-01` |
| `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Optional non-canonical restaurant label. | `Kathmandu Branch` |

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:29-61

## Restaurant Attribution Source Of Truth

For outlet-owned RestroX:

```txt
Webhook Token
-> PosIntegration
-> Outlet
-> Bound Restaurant
```

Restaurant identity is resolved from:

- `integration.external_location_id`
- `integration.external_location_name`

not from webhook payload fields.

## Ignored By Samparka If Present

Any fields outside the parser mappings above are not required for the canonical webhook flow.
Webhook payload restaurant identity fields are optional, non-canonical metadata for outlet-owned integrations.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
