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
