---
title: Final Verification Report
description: Review the backend-driven contract, schema, and content verification for the POS partner guide package.
sidebarTitle: Verification Report
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
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

- Connect accepts `integrationKey`, `externalLocationId`, and optional `externalLocationName` as the primary location-binding contract.
- Connect returns only `success`, `integrationId`, `token`, and `status`.
- Test sale wraps the inner webhook response inside `data` and does not return the raw webhook response directly.
- Customer search returns `exists: false` or `exists: true` plus one serialized `customer`.
- Customer detail returns `{ customer: ... }`.
- Webhook success and error bodies are minimal `{ success, message }` envelopes with scenario-specific status codes and exact message text.

## Implementation / Documentation Mismatches Corrected

- Partner API examples now use canonical `Authorization: Bearer {{providerApiKey}}` authentication instead of `x-partner-key` as the primary contract.
- Connect docs now use `externalLocationId` and `externalLocationName` as the primary request fields, with singular `restaurantId` and `restaurantName` kept only as backward-compatible aliases.
- Connect docs now show the exact success payload and explicitly avoid removed fields such as `message`, `restaurantId`, and `externalLocationId`.
- Webhook setup docs now tell integrators to construct `/webhook/{provider}/{token}` from the returned `token`.
- Test-sale examples documented raw webhook responses instead of the actual wrapped `responseHandler.success` envelope.
- Customer search docs and examples still used an old array-style response shape instead of `{ exists, customer }`.
- Customer detail docs pointed to merchant-style customer routes and merchant-style `data` envelopes instead of the implemented partner route and `{ customer }` shape.
- Webhook docs did not consistently document the exact `409` responses for missing binding and disconnected integration.
- Some example material still implied payload restaurant fields were required for outlet-owned webhook routing, but the implementation resolves location identity from the integration bound to the token.
- Partner-facing docs contained internal `Source:` citations and backend file paths that should not appear in published integration guidance.

## Search Verification

Reviewed repository-wide references for:

- `^Source:`
- legacy customer-search routes
- legacy partner-key auth examples
- webhook wording that treats payload location fields as required

Result:

- Active POS customer-search examples now use `/api/partners/{provider}/customers/search`.
- Active POS customer-detail examples now use `/api/partners/{provider}/customers/{customerId}`.
- Active partner API examples now use `Authorization: Bearer {{providerApiKey}}` as the canonical auth model.
- Active POS webhook docs now use the exact backend status codes and exact message text for the documented scenarios.
- Active POS onboarding docs now show `token` returned by connect and use it directly for webhook setup.

## Remaining Documentation Risks

- Customer detail verification proves the partner-customer contract and the stored customer state, but loyalty transaction inspection still depends on merchant-side tooling outside the partner HTTP response itself.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

## Purchase QR Verification

The Purchase QR endpoint has been verified against the backend implementation:

- `POST /integrations/pos/{provider}/purchase-qr` accepts `amount`, `currency`, and `items` with `X-Integration-Key` in the request body.
- Auth is the provider API key (`Authorization: Bearer <provider_api_key>`) plus the integration key.
- Success response returns `qr_link`, `purchase_reference`, `amount`, `currency`, `status`, and `expires_at`.
- `409 pos_not_connected` when the POS integration is not `CONNECTED`/`ACTIVE`.
- `422 no active connected communication provider` when no WhatsApp/Wapio provider is active.
- `400` validation errors for missing/invalid `amount`, `items`, or `currency`.
- `401` for missing/invalid provider API key or integration key.
- `409 already processed` for completed/claimed sessions.
- `410 expired` for expired sessions.
- `502 QR generation failed` when Wapio/redirect fails.

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
  <Card title="Testing & Verification" icon="flask-conical" href="/integrations/pos/purchase-qr/testing">
    Automated test cases and deployment notes.
  </Card>
</Columns>
  </Tab>
</Tabs>
