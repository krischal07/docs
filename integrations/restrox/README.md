---
title: RestroX Partner Guide
description: Canonical overview of the RestroX connect and webhook integration with Samparka Loyalty.
sidebarTitle: Guide Overview
---

# RestroX Partner Guide

Samparka is a customer loyalty platform. The RestroX integration has two distinct stages:

1. Connect one RestroX restaurant to one outlet-owned Samparka integration with `POST /api/partners/restrox/connect`.
2. Send sale, refund, and void webhooks to the token-based webhook URL after the integration is connected.

Source:
samparka-backend/src/index.js:146-155
samparka-backend/src/integrations/pos/partners/restrox/service.js:116-260
samparka-backend/src/integrations/pos/routes.js:11-15
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-30

## Integration Flow

1. Receive the Samparka Integration Key for the outlet-owned RestroX integration.
2. Call `POST /api/partners/restrox/connect` with `integrationKey`, `restaurantId`, and optional `restaurantName`.
3. Samparka resolves `PosIntegration`, validates `ownership_mode = OUTLET_OWNED`, and persists `external_location_id`.
4. Confirm the response returns `status = CONNECTED`.
5. Send a test `order.completed` webhook to `/webhook/restrox/{token}`.
6. Confirm a `200` acknowledgment.
7. Repost the same sale payload once to confirm duplicate safety.
8. Send a `refund.created` webhook that references the original sale identifier.
9. Complete the go-live checklist.

Source:
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38

## Quick Links

- [Partner Handoff](./PARTNER-HANDOFF)
- [Quick Start](./quick-start)
- [Authentication](./authentication)
- [Endpoint Catalog](./endpoint-catalog)
- [Webhook Endpoint](./webhook-endpoint)
- [Event Types](./event-types)
- [Payload Reference](./payload-reference)
- [Response Reference](./response-reference)
- [Refunds](./refunds)
- [Idempotency](./idempotency)
- [Location Mapping](./location-mapping)
- [Testing Guide](./testing-guide)
- [Troubleshooting](./troubleshooting)
- [Integration Checklist](./integration-checklist)
- [OpenAPI](./openapi.yaml)
- [Postman Collection](./postman-collection.json)

## Breaking Change Notice

The old array-based connect payload is intentionally no longer supported.

No longer supported:

```json
{
  "integrationKey": "...",
  "account": {
    "id": "...",
    "name": "..."
  },
  "locations": [
    {
      "restaurantId": "...",
      "restaurantName": "..."
    }
  ]
}
```

Replacement:

```json
{
  "integrationKey": "...",
  "restaurantId": "...",
  "restaurantName": "..."
}
```

## Important Delivery Note

A `200 Event received` response means Samparka accepted the webhook delivery. It does not guarantee that loyalty activity was created, because customer and restaurant requirements are checked after the request is accepted.

Source:
samparka-backend/src/integrations/pos/partners/restrox/controller.js:7-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/controller.js:332-365
samparka-backend/src/loyalty/handlers/saleCompletedHandler.js:101-110
samparka-backend/src/loyalty/handlers/saleCompletedHandler.js:134-142
