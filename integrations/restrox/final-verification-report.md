---
title: Final Verification Report
description: Review the contract, schema, and content verification for the RestroX partner guide package.
sidebarTitle: Verification Report
---

# Final Verification Report

This report is the acceptance gate for the RestroX partner package after the outlet-owned connect cleanup.

## Files Updated

- `samparka-backend/src/integrations/pos/partners/restrox/controller.js`
- `samparka-backend/src/integrations/pos/partners/restrox/service.js`
- `samparka-backend/__tests__/integrations/partner.routes.test.js`
- `samparka-backend/src/integrations/pos/merchant/constants.js`
- `samparka-frontend/src/components/integrations/IntegrationKeyCard.jsx`
- `docs/restrox-partner-guide/README.md`
- `docs/restrox-partner-guide/PARTNER-HANDOFF.md`
- `docs/restrox-partner-guide/quick-start.md`
- `docs/restrox-partner-guide/payload-reference.md`
- `docs/restrox-partner-guide/location-mapping.md`
- `docs/restrox-partner-guide/testing-guide.md`
- `docs/restrox-partner-guide/troubleshooting.md`
- `docs/restrox-partner-guide/endpoint-catalog.md`
- `docs/restrox-partner-guide/reference/postman-collection.mdx`
- `docs/restrox-partner-guide/openapi.yaml`
- `docs/restrox-partner-guide/postman-collection.json`
- `docs/restrox-partner-guide/examples/payloads.json`

## Connect Payloads Rewritten

Active connect contract:

```json
{
  "integrationKey": "string",
  "restaurantId": "string",
  "restaurantName": "string"
}
```

Removed from the active connect contract:

- `account`
- `locations`
- `locationList`
- `restaurantList`
- `branches`

## Postman Requests Updated

- Added `Connect Restaurant`
- Added `Reject Legacy Connect Payload`
- Added assertions for `status = CONNECTED`
- Added assertions for persisted `externalLocationId`

## OpenAPI Examples Updated

- Added `POST /api/partners/restrox/connect`
- Added singular connect request example
- Added connect success example
- Added connect validation failure example

## OpenAPI Schemas Updated

- Added `RestroxConnectRequest`
- Added `RestroxConnectResponse`
- Confirmed `integrationKey` required
- Confirmed `restaurantId` required
- Confirmed `restaurantName` optional
- Confirmed the connect schema does not expose `account`, `locations`, `locationList`, `restaurantList`, or `branches`

## Diagrams Updated

Connect sequence is now:

```txt
Connect
-> Bind Restaurant
-> CONNECTED
-> Receive Sale
-> ACTIVE
```

## Legacy Persistence Review

- `partner_metadata.accountId`
  - action taken: removed from the active connect path
  - reason: the singular contract no longer accepts account metadata
- `partner_metadata.accountName`
  - action taken: removed from the active connect path
  - reason: the singular contract no longer accepts account metadata
- `external_location_id`
  - action taken: retained
  - reason: it is the required outlet-owned restaurant binding
- `external_location_name`
  - action taken: retained
  - reason: it stores the optional display label for the bound restaurant
- `partner_metadata.reviewIssues`
  - action taken: retained
  - reason: this metadata is still used by merchant readiness flows outside the active RestroX connect path
- `posIntegrationLocation` mapping rows
  - action taken: retained
  - reason: webhook routing and non-connect merchant tooling still depend on these documents outside the active connect path

## Implementation / Documentation Mismatches

- None after this cleanup.

## Remaining Connect-Flow Risks

- `POST /api/partners/restrox/sync-locations` still exists for compatibility, so future changes must avoid reintroducing it into onboarding or partner-facing connect guidance.
- Merchant readiness code still contains older review and mapping language for broader POS flows; only the outlet-owned RestroX connect path was normalized here.

## Contract Removal Verification

Search targets reviewed in RestroX docs and artifacts:

- `locations[`
- `restaurantList`
- `locationList`
- `branches`

Result:

- No active RestroX connect contract requires or documents array-based restaurant selection.
