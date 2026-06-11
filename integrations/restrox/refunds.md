---
title: Refunds
description: Send refund events correctly so Samparka can reverse prior loyalty activity.
sidebarTitle: Refunds
---

Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should reference the original sale identifier that was already sent to Samparka.

See also: [Event Types](./event-types) and [Testing Guide](./testing-guide).

## What RestroX Needs To Send

Send:

1. `event_type` as `refund.created` or `order.voided`
2. The **exact same** sale identifier in `order_id`, `transaction_id`, or `id` that was used in the original sale event
3. A location identifier such as `restaurantId`

The refund and the original sale must use the same identifier field. If the original sale used `order_id`, the refund must also use `order_id` with the same value. Samparka uses this identifier as part of the idempotency key (`{event_type}:{order_id}`) — the refund event type prefix (`refund.created:`) distinguishes it from the original sale.

## What Response To Expect

A valid refund or void delivery can return:

```json
{
  "success": true,
  "message": "Event received"
}
```

If the same refund is resent, Samparka can return:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## When No Original Sale Is Found

Samparka always returns HTTP 200 as a delivery acknowledgement for refund and void events, even when no original sale can be matched. The HTTP 200 response only confirms the event was received — it does not confirm the reversal succeeded.

When no matching original sale exists, the internal processing result is `failed_terminal` with reason `original_event_not_found`. No loyalty points are reversed in this case.

To ensure refund matching succeeds, the refund event must supply the same order identifier (`order_id`, `transaction_id`, or `id`) that was used in the original sale event.

## What To Do If Something Goes Wrong

If a refund does not produce the expected reversal, verify that the refund uses the same original sale identifier that was sent in the sale webhook. A `200` acknowledgement does not confirm loyalty was reversed.
