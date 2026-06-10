---
title: Event Types
description: Review the RestroX webhook events supported by Samparka Loyalty.
sidebarTitle: Event Types
---

Samparka’s recommended RestroX event types are:

See also: [Payload Reference](./payload-reference).

- `order.completed` for completed sales
- `refund.created` for refunds
- `order.voided` for voided or canceled sales

## `order.completed`

Purpose: send a completed sale that may qualify for loyalty processing.

Required fields: `event_type`, one sale identifier, one location identifier.

Example payload: use `sale_completed_request` from [`examples/payloads.json`](./examples/payloads.json).

Example response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## `refund.created`

Purpose: send a refund for a previously reported sale.

Required fields: `event_type`, the original sale identifier, one location identifier.

Example payload: use `refund_created_request` from [`examples/payloads.json`](./examples/payloads.json).

Example response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## `order.voided`

Purpose: send a voided sale when the original sale should no longer count.

Required fields: `event_type`, the original sale identifier, one location identifier.

Example payload: use `voided_sale_request` from [`examples/payloads.json`](./examples/payloads.json).

Example response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## Accepted Aliases

Samparka maps these additional event type strings to the same business-domain events. Prefer the canonical names above in new integrations, but all aliases below are accepted:

| Sent by RestroX | Maps to | Notes |
| --------------- | ------- | ----- |
| `order.completed` | `sale.completed` | Canonical recommended name |
| `transaction.completed` | `sale.completed` | Accepted alias |
| `sale.completed` | `sale.completed` | Accepted alias |
| `order.placed` | `sale.completed` | Accepted alias |
| `refund.created` | `refund.created` | Canonical recommended name |
| `order.refund` | `refund.created` | Accepted alias |
| `refund.issued` | `refund.created` | Accepted alias |
| `order.voided` | `sale.voided` | Canonical recommended name |
| `order.void` | `sale.voided` | Accepted alias |
| `order.cancelled` | `sale.voided` | Accepted alias |
| `order.canceled` | `sale.voided` | Accepted alias |

## Unknown Event Types

If RestroX sends an `event_type` that does not match any entry in the table above, Samparka silently maps it to `sale.completed` and processes it as a completed sale. No error is returned. Verify your `event_type` values match the table to ensure correct processing.
