---
title: Payload Reference
description: Required and optional POS fields for connect, test-sale, and webhook delivery.
sidebarTitle: Payload Reference
---


Samparka reads different fields for `connect`, `test-sale`, and direct webhook delivery.

<Info>
**Also see — Purchase QR request fields** (`amount`, `currency`, `items`): [Request / Response Contract](/integrations/pos/purchase-qr/request-response).
</Info>

## Connect Request

`POST /api/partners/{provider}/connect`

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `PosIntegration`. | `pos-key-001` |
| `restaurantId` | string | Yes | POS restaurant identifier persisted as `external_location_id`. | `ktm-branch-01` |

### Optional Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `restaurantName` | string | No | Human-readable restaurant name persisted as `external_location_name`. | `Kathmandu Branch` |

## Test Sale Request

`POST /api/partners/{provider}/test-sale`

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `PosIntegration`. | `pos-key-001` |
| `payload` | object | Yes | Webhook-shaped event body submitted into the webhook processing pipeline. | See webhook fields below. |

## Webhook Fields

`POST /webhook/{provider}/{token}`

### Required Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` or `type` | string | Yes | POS event name. | `order.completed` |

### Optional Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `order_id` or `transaction_id` or `id` | string | No | Provider transaction identifier used for transaction reference and idempotency. | `pos-sale-1001` |
| `created_at` or `timestamp` | string | No | Event time from POS. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Primary customer identifier for search and loyalty attribution. | `9800000101` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `restaurantId` or `restaurant_id` or `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | No | Optional non-canonical restaurant metadata. Outlet-owned attribution is resolved from the integration binding instead. | `ktm-branch-01` |
| `restaurantName` or `restaurant_name` or `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Optional non-canonical restaurant label. | `Kathmandu Branch` |

## Purchase QR Request

`POST /integrations/pos/{provider}/purchase-qr`

The POS sends the cash-sale details and receives a scannable purchase QR `qr_link` in response. Auth is the provider API key (`Authorization: Bearer <provider_api_key>`) plus the integration key (`X-Integration-Key`). No `webhook_token` in the path. See [Purchase QR Request / Response](./purchase-qr/request-response).

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `amount` | number | Yes | Sale amount. Must be greater than `0`. | `1250` |
| `items` | array | Yes | Product items in the bill. Each item has `name`, `qty`, and `price`. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |

### Optional Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `currency` | string | No | 3-letter currency code. Defaults to `NPR` if omitted. | `NPR` |

<Note>
  `items` is **required** for purchase QR because the POS must provide the product items in the bill for loyalty attribution.
</Note>

## Restaurant Attribution Source Of Truth

For outlet-owned POS:

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

## Additional Notes

- Webhook payload restaurant fields are optional, non-canonical metadata for outlet-owned integrations.
- Fields outside the parser mappings are not required for the canonical partner flow.
