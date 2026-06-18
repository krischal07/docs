---
title: Final Verification Report
description: Review the backend-driven contract, schema, and content verification for the POS partner guide package.
sidebarTitle: Verification Report
---

# Final Verification Report

This report captures the backend-driven correction of the POS partner documentation package against `../samparka_vps/samparka-backend`.

## Files Updated

- `README.md`
- `PARTNER-HANDOFF.md`
- `quick-start.md`
- `testing-guide.md`
- `payload-reference.md`
- `response-reference.md`
- `authentication.md`
- `endpoint-catalog.md`
- `troubleshooting.md`
- `openapi.yaml`
- `postman-collection.json`
- `examples/payloads.json`
- `examples/sale-completed.md`
- `examples/refund-created.md`
- `examples/duplicate-webhook.md`
- `examples/blocked-location.md`
- mirrored files under `integrations/pos/`

## Contract Summary

Verified partner-facing contracts:

- Connect returns a custom top-level response including `token` and `restaurantId`.
- Test sale wraps the inner webhook response inside `data` and does not return the raw webhook response directly.
- Customer search returns `exists: false` or `exists: true` plus one serialized `customer`.
- Customer detail returns `{ customer: ... }`.
- Webhook success and error bodies are minimal `{ success, message }` envelopes with scenario-specific status codes and exact message text.

## Implementation / Documentation Mismatches Found

- Connect docs omitted `token` and `restaurantId` from successful responses.
- Connect docs did not describe the optional `idempotent: true` field for same-binding reconnects.
- Webhook setup docs implied the token had to be discovered elsewhere, even though connect already returns it.
- Test-sale examples documented raw webhook responses instead of the actual wrapped `responseHandler.success` envelope.
- Customer search docs and examples still used an old array-style response shape instead of `{ exists, customer }`.
- Customer detail docs pointed to merchant-style customer routes and merchant-style `data` envelopes instead of the implemented partner route and `{ customer }` shape.
- Webhook docs did not consistently document the exact `409` responses for missing binding and disconnected integration.
- Some example material still implied payload restaurant fields were required for outlet-owned webhook routing, but the implementation resolves restaurant identity from the integration bound to the token.
- Partner-facing docs contained internal `Source:` citations and backend file paths that should not appear in published integration guidance.

## Search Verification

Reviewed repository-wide references for:

- `^Source:`
- legacy customer-search routes
- legacy merchant bearer-auth customer-search examples
- webhook wording that treats restaurant payload fields as required

Result:

- Active POS customer-search examples now use `/api/partners/{provider}/customers/search`.
- Active POS customer-detail examples now use `/api/partners/{provider}/customers/{customerId}`.
- Active POS webhook docs now use the exact backend status codes and exact message text for the documented scenarios.
- Active POS onboarding docs now show `token` returned by connect and use it directly for webhook setup.

## Remaining Documentation Risks

- Customer detail verification proves the partner-customer contract and the stored customer state, but loyalty transaction inspection still depends on merchant-side tooling outside the partner HTTP response itself.
