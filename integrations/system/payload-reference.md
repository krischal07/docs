---
title: Payload Reference
description: Required and optional System fields for connect, test-sale, and webhook delivery.
sidebarTitle: Payload Reference
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">

Samparka reads different fields for `connect`, `test-sale`, and direct webhook delivery.

<Info>
**Also see — Purchase QR request fields** (`amount`, `currency`, `items`): [Request / Response Contract](/integrations/system/purchase-qr/request-response).
</Info>

## Connect Request

`POST /api/partners/{provider}/connect`

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `SystemIntegration`. | `system-key-001` |
| `restaurantId` | string | Yes | System restaurant identifier persisted as `external_location_id`. | `ktm-branch-01` |

### Optional Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `restaurantName` | string | No | Human-readable restaurant name persisted as `external_location_name`. | `Kathmandu Branch` |

## Test Sale Request

`POST /api/partners/{provider}/test-sale`

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `SystemIntegration`. | `system-key-001` |
| `payload` | object | Yes | Webhook-shaped event body submitted into the webhook processing pipeline. | See webhook fields below. |

## Webhook Fields

`POST /webhook/{provider}/{token}`

### Required Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` or `type` | string | Yes | System event name. | `order.completed` |

### Optional Fields

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `order_id` or `transaction_id` or `id` | string | No | Provider transaction identifier used for transaction reference and idempotency. | `system-sale-1001` |
| `created_at` or `timestamp` | string | No | Event time from System. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Primary customer identifier for search and loyalty attribution. | `9800000101` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `restaurantId` or `restaurant_id` or `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | No | Optional non-canonical restaurant metadata. Outlet-owned attribution is resolved from the integration binding instead. | `ktm-branch-01` |
| `restaurantName` or `restaurant_name` or `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Optional non-canonical restaurant label. | `Kathmandu Branch` |

## Purchase QR Request

`POST /integrations/system/{provider}/purchase-qr`

The System sends a webhook-style envelope with the cash-sale details and receives a scannable purchase QR `qr_link` in response. Auth is the provider API key (`Authorization: Bearer <provider_api_key>`); the store is resolved from the API key's `store_id` scope. The store's integration key travels in the request body (`integrationKey`), **not a header**. No `webhook_token` or `X-Integration-Key` header in the path. See [Purchase QR Request / Response](./purchase-qr/request-response).

### Required Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Store's integration key. Must match the store the provider API key is scoped to, else `401`. Sent in the **request body**, not a header. | `SPK-RX-TTMFHBYZ` |
| `payload` | object | Yes | Wrapper object holding the sale details. | — |
| `payload.event_type` | string | Yes | Must be `"order.completed"`, else `400`. | `order.completed` |
| `payload.amount` | number | Yes | Sale amount. Must be greater than `0`. | `1250` |
| `payload.items` | array | Yes | Product items in the bill. Each item has `name`, `qty`, and `price`. Σ(items) must equal amount ±0.01, else `400`. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |

### Optional Properties

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `payload.order_id` | string | Recommended | POS order/receipt id (max 100 chars). Enables safe retries — see [Idempotency](#idempotency). Can also be sent as an `Idempotency-Key` header; when both are present they must match. | `ORDER-1001` |
| `payload.created_at` | string | No | ISO 8601 timestamp of the sale; stored in session metadata. Invalid dates → `400`. | `2026-06-08T10:15:00.000Z` |
| `payload.currency` | string | No | 3-letter currency code. Defaults to `NPR` if omitted. | `NPR` |

<Note>
  `items` is **required** for purchase QR because the System must provide the product items in the bill for loyalty attribution.
  `payload.event_type` must be `"order.completed"` — other values return `400`.
  `integrationKey` is validated against the store resolved from the API key's `store_id` scope — mismatch returns `401`.
  **No `customer_phone` field is accepted on this endpoint** — the endpoint is phone-less. Customer identity is resolved during the WhatsApp claim flow.
</Note>

## Restaurant Attribution Source Of Truth

For outlet-owned System:

```txt
Webhook Token
-> SystemIntegration
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
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

`POST /integrations/system/{provider}/purchase-qr`

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `amount` | number | Yes | Transaction amount, must be greater than 0. | `1250` |
| `currency` | string | No | 3-letter currency code. Defaults to `NPR` if omitted. | `NPR` |
| `items` | array | Yes | Product items in the bill. Each item has `name`, `qty`, and `price`. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |

See [Purchase QR Request / Response](/integrations/system/purchase-qr/request-response) for the full contract.

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
  <Card title="Integration Guide" icon="rocket" href="/integrations/system/purchase-qr/integration-guide">
    Example curl calls and behavior matrix.
  </Card>
</Columns>
  </Tab>
</Tabs>
