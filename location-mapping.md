---
title: Restaurant Binding Reference
description: Understand how the connected restaurant binding is used during webhook delivery.
sidebarTitle: Restaurant Binding
---

# Restaurant Binding Reference

RestroX connect is outlet-owned and singular:

```txt
One Integration
=
One Outlet
=
One Restaurant
```

The `connect` step persists:

- `external_location_id`
- `external_location_name`

Webhook delivery then resolves the bound restaurant from the integration that owns the webhook token.

Source:
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/controller.js:285-365

## Connect Source Of Truth

```txt
integrationKey
-> restaurantId
-> external_location_id
```

There is no legacy multi-step restaurant onboarding step in the active connect path.

## Canonical Webhook Attribution

```txt
Webhook Token
-> PosIntegration
-> Bound Restaurant
```

Restaurant identity is derived from the integration binding.

## Optional Payload Restaurant Fields

Webhook payloads may still include:

- `external_location_id`
- `location_id`
- `outlet_id`
- `branch_id`
- `external_location_name`
- `location_name`
- `branch_name`
- `outlet_name`

These fields are optional, non-canonical metadata for outlet-owned integrations.

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:43-54

## What Response To Expect

If the token is valid, Samparka can still return `200 Event received` even when the integration is not yet in a processable state. That acknowledgment means the delivery was accepted, not necessarily credited.

Source:
samparka-backend/src/integrations/pos/controller.js:246-349
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77

## What To Do If Something Goes Wrong

If a sale is accepted but expected loyalty activity is missing, verify:

1. The integration was connected before the webhook was sent.
2. The integration still has a bound restaurant.
3. The payload includes a valid customer phone and transaction reference.

Source:
samparka-backend/src/integrations/pos/controller.js:285-365
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77
