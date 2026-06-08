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

## Event Delivery

| Event Type | Tested | Result |
| ---------- | ------ | ------ |
| `order.completed` |  |  |
| `refund.created` |  |  |
| `order.voided` |  |  |

## Location Mapping

- [ ] `external_location_id` matches configured outlet
- [ ] Unknown location tested
- [ ] Blocked-location scenario verified

## Retry Behavior

- [ ] Same payload sent twice
- [ ] Duplicate acknowledged safely
- [ ] No unexpected duplicate outcome observed

## Refund Validation

- [ ] Refund references original sale
- [ ] Refund test completed
- [ ] Duplicate refund repost tested

## Production Readiness

- [ ] Production webhook URL configured
- [ ] Outlet mappings verified
- [ ] Token stored securely
- [ ] Monitoring enabled

## Go-Live Signoff

Partner Representative:  
Date:

Samparka Representative:  
Date:

Status:
- [ ] Approved
- [ ] Pending
- [ ] Blocked
