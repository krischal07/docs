---
title: Integration Checklist
description: Final go-live checklist for validating a RestroX to Samparka integration.
sidebarTitle: Integration Checklist
---

# Integration Checklist

See also: [Testing Guide](./testing-guide).

## Connect Contract

- [ ] `POST /api/partners/restrox/connect` tested
- [ ] `integrationKey` verified
- [ ] `restaurantId` verified
- [ ] Bound restaurant stored on the integration
- [ ] Response status is `CONNECTED`
- [ ] Only the singular connect payload is used in active tooling and tests

Source:
samparka-backend/src/index.js:146-155
samparka-backend/src/integrations/pos/partners/restrox/controller.js:9-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260

## Webhook Connectivity

- [ ] Webhook URL configured correctly
- [ ] HTTPS enabled
- [ ] Correct token used
- [ ] Test webhook reaches Samparka
- [ ] Invalid token test verified

Source:
samparka-backend/src/index.js:151-155
samparka-backend/src/integrations/pos/controller.js:200-205

## Event Delivery

| Event Type | Tested | Result |
| ---------- | ------ | ------ |
| `order.completed` |  |  |
| `refund.created` |  |  |
| `order.voided` |  |  |

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-30

## Business Outcome Verification

- [ ] First valid sale moves the integration to `ACTIVE`
- [ ] Customer can be found by the phone used in the test sale
- [ ] Customer belongs to the expected store or business
- [ ] Customer detail response includes loyalty data
- [ ] Loyalty transaction exists for the verified sale
- [ ] Points awarded reflect successful sale processing

## Restaurant Binding

- [ ] Restaurant binding was established through connect
- [ ] Integration shows the expected `externalLocationId`
- [ ] Missing-binding state verified before connect

Source:
samparka-backend/src/integrations/pos/controller.js:285-365
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77

## Retry Behavior

- [ ] Same payload sent twice
- [ ] Duplicate acknowledged safely
- [ ] No unexpected duplicate outcome observed

Source:
samparka-backend/src/integrations/pos/controller.js:293-312

## Refund Validation

- [ ] Refund references original sale
- [ ] Refund test completed
- [ ] Duplicate refund repost tested

Source:
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:158-160

## Production Readiness

- [ ] Production webhook URL configured
- [ ] Connected restaurant binding verified
- [ ] Token stored securely
- [ ] Monitoring enabled
- [ ] Customer verification and points verification completed

Source:
samparka-backend/src/integrations/pos/controller.js:285-365

## Go-Live Signoff

Partner Representative:  
Date:

Samparka Representative:  
Date:

Status:
- [ ] Approved
- [ ] Pending
- [ ] Blocked
