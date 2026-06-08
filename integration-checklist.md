---
title: Integration Checklist
description: Final go-live checklist for validating a RestroX to Samparka integration.
sidebarTitle: Integration Checklist
---

# Integration Checklist

See also: [Testing Guide](./testing-guide).

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

## Location Mapping

- [ ] `external_location_id` matches configured outlet
- [ ] Unknown location tested
- [ ] Blocked-location scenario verified

Source:
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77
samparka-backend/src/models/posIntegrationLocationModel.js:25-30

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
- [ ] Outlet mappings verified
- [ ] Token stored securely
- [ ] Monitoring enabled

Source:
samparka-backend/src/integrations/pos/locationResolutionService.js:9-27
samparka-backend/src/models/posIntegrationLocationModel.js:35-40

## Go-Live Signoff

Partner Representative:  
Date:

Samparka Representative:  
Date:

Status:
- [ ] Approved
- [ ] Pending
- [ ] Blocked
