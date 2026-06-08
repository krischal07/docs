---
title: Event Types
description: Review the RestroX webhook events supported by Samparka Loyalty.
sidebarTitle: Event Types
---

# Event Types

Samparka’s recommended RestroX event types are:

See also: [Payload Reference](./payload-reference).

- `order.completed` for completed sales
- `refund.created` for refunds
- `order.voided` for voided or canceled sales

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-30

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

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
samparka-backend/src/integrations/pos/controller.js:351-365

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

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:27-29
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38
samparka-backend/src/integrations/pos/controller.js:351-365

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

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:23-26
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38
samparka-backend/src/integrations/pos/controller.js:351-365
