---
title: Integration Checklist
description: Final go-live checklist for validating a RestroX to Samparka integration.
sidebarTitle: Integration Checklist
---

See also: [Testing Guide](./testing-guide) and [Readiness Checklist](./native/readiness-checklist).

## Native Onboarding

- [ ] Connection Key received from merchant's Samparka integration setup
- [ ] `/api/partners/restrox/connect` called successfully with merchant's Connection Key
- [ ] Location sync completed (`/api/partners/restrox/sync-locations`)
- [ ] `reviewRequired: false` confirmed in sync response (or review issues resolved manually)
- [ ] Integration status advanced past `AWAITING_CONNECTION`

## Customer Lookup

- [ ] Customer search tested via `/api/partners/restrox/customers/search?phone=...`
- [ ] `exists: true` and `exists: false` responses both verified
- [ ] `x-integration-key` header confirmed present

## Native Test Sale

- [ ] `/api/partners/restrox/test-sale` called with a valid payload
- [ ] Response: `{ "success": true, "message": "Test sale submitted", "data": { "success": true, "message": "Event received" } }`
- [ ] Integration status confirmed as `READY` after test sale

## Webhook Connectivity

- [ ] Webhook URL format: `POST /webhook/restrox/{token}`
- [ ] HTTPS enabled
- [ ] Correct per-location token used
- [ ] Direct test webhook reaches Samparka
- [ ] Invalid token test verified (`404 Invalid webhook token`)

## Event Delivery

| Event Type | Tested | Result |
| ---------- | ------ | ------ |
| `order.completed` |  |  |
| `refund.created` |  |  |
| `order.voided` |  |  |

## Location Mapping

- [ ] `external_location_id` in payloads matches the value used during location sync
- [ ] Unknown location tested via direct webhook — confirmed returns `200 Event received` (blocked internally, no loyalty)
- [ ] Confirmed the difference: test-sale with unknown location returns `404`, direct webhook returns `200`

## Idempotency

- [ ] Same payload sent twice
- [ ] Duplicate acknowledged safely (`200 Event already processed`)
- [ ] Confirmed sale and refund for the same `order_id` are processed as separate events (different `event_type` prefix)

## Refund Validation

- [ ] Refund uses same `order_id` / `transaction_id` / `id` as original sale
- [ ] Refund test completed
- [ ] Duplicate refund repost tested

## Production Readiness

- [ ] Production webhook URL configured
- [ ] Outlet mappings verified — all locations `active`, none `STALE`
- [ ] Token stored securely
- [ ] Connection Key stored securely (not exposed to end users)
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
