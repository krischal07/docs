---
title: Payload Reference
description: Required, optional, and ignored RestroX payload fields for Samparka webhook delivery.
sidebarTitle: Payload Reference
---

Samparka reads the fields below from RestroX webhooks.

## Required Fields

Only one field is strictly required at the HTTP level. Its absence causes a `400` response:

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` | string | **Yes** | RestroX event type. `type` is also accepted if `event_type` is absent. | `order.completed` |

## Strongly Recommended Fields

These fields are not HTTP-level required — their absence does not cause a `400` error — but omitting them prevents correct loyalty processing:

| Field | Type | Notes | Example |
| ----- | ---- | ----- | ------- |
| `order_id` or `transaction_id` or `id` | string | Transaction reference. Without this Samparka falls back to a SHA-256 idempotency key derived from the payload. Idempotent retry behavior depends on the same identifier being resent. | `restrox-sale-1001` |
| `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | Outlet identifier used for location resolution. Without this the event is stored with `blocked_unmapped_location` status and no loyalty is awarded. Samparka returns `200 Event received` regardless. | `ktm-branch-01` |

## Optional Fields

| Field | Type | Description | Example |
| ----- | ---- | ----------- | ------- |
| `created_at` or `timestamp` | string | Event time from RestroX. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | Transaction amount. Defaults to `0` if omitted. | `850` |
| `currency` or `currency_code` | string | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | Customer phone number for loyalty association. | `9800000101` |
| `items` or `line_items` | array | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | Human-readable outlet name. Used for display and auto-matching during location sync. | `Kathmandu Branch` |

## Ignored By Samparka If Present

Any fields outside the parser mappings above are passed through to metadata but do not affect processing.
