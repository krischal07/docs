---
title: Payload Reference
description: Required and optional POS fields for connect, test-sale, and webhook delivery.
sidebarTitle: Payload Reference
---


Samparka reads different fields for `connect`, `test-sale`, and direct webhook delivery.

<Info>
**Also see — Purchase QR request fields** (`bill_id`, `amount`, `currency`, `customer_phone`): [Request / Response Contract](/integrations/pos/purchase-qr/request-response).
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

`POST /integrations/pos/{provider}/{token}/purchase-qr`

The POS sends the cash-sale details and receives a scannable purchase QR `qr_link` in response. See [Purchase QR Request / Response](./purchase-qr/request-response).

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `bill_id` | string | Yes | Unique bill identifier. Used as part of the idempotency key `posqr:{provider}:{bill_id}`. | `INV-2041` |
| `amount` | number | Yes | Sale amount. Must be greater than `0`. | `1250` |
| `customer_phone` | string | Yes | Customer phone, matches `/^\+?[0-9]{7,15}$/`. Used as the QR deep-link target and loyalty attribution. | `+9779800001234` |

### Optional Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `currency` | string | No | 3-letter currency code. Defaults to `NPR` if omitted. | `NPR` |

<Note>
  `customer_phone` is **required** for purchase QR because the receipt QR is a WhatsApp-style deep link — without a valid target number there is no usable QR. This intentionally differs from the customer lookup flow, where a missing or invalid phone never blocks checkout.
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
