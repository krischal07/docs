---
title: Refunds
description: Send refund events correctly so Samparka can reverse prior loyalty activity.
sidebarTitle: Refunds
---

# Refunds

Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should reference the original sale identifier that was already sent to Samparka.

See also: [Event Types](./event-types) and [Testing Guide](./testing-guide).

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:23-29
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38

## What RestroX Needs To Send

Send:

1. `event_type` as `refund.created` or `order.voided`
2. The original sale identifier in `order_id`, `transaction_id`, or `id`
3. Customer and amount data needed for the reversal workflow

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:20-27
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:23-29

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

Source:
samparka-backend/src/integrations/pos/controller.js:301-365
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:158-160

## What To Do If Something Goes Wrong

If a refund does not produce the expected result, verify that the refund uses the same original sale identifier that was sent in the sale webhook. If Samparka cannot match the original sale identifier, the reversal cannot proceed.

Source:
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38
