---
title: Refunds
description: Send refund events correctly so Samparka can reverse prior loyalty activity.
sidebarTitle: Refunds
---

# Refunds

Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should reference the original sale identifier that was already sent to Samparka.

See also: [Event Types](./event-types) and [Testing Guide](./testing-guide).

## What RestroX Needs To Send

Send:

1. `event_type` as `refund.created` or `order.voided`
2. The original sale identifier in `order_id`, `transaction_id`, or `id`
3. A location identifier such as `external_location_id`

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

## What To Do If Something Goes Wrong

If a refund does not produce the expected result, verify that the refund uses the same original sale identifier that was sent in the sale webhook. If Samparka cannot match the original sale identifier, the reversal cannot proceed.
