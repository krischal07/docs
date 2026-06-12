---
title: RestroX Partner Handoff
description: Canonical partner handoff summary for RestroX onboarding with Samparka Loyalty.
sidebarTitle: Partner Handoff Source
---

# RestroX Partner Handoff

This package is the shareable entry point for the RestroX connect and webhook integration with Samparka. Connect uses a singular restaurant-binding contract. The active RestroX path does not use any legacy multi-step restaurant onboarding or payload-side attribution checks.

Source:
samparka-backend/src/index.js:146-155
samparka-backend/src/integrations/pos/partners/restrox/controller.js:7-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/routes.js:11-15

## Start Here

1. [Overview](./README)
2. [Quick Start](./quick-start)
3. [Endpoint Catalog](./endpoint-catalog)
4. [Payload Reference](./payload-reference)
5. [Testing Guide](./testing-guide)

## Integration Flow

```mermaid
flowchart TD
  A["Merchant Creates Integration"] --> B["Partner Connects Restaurant"]
  B --> C["System Stores Restaurant Binding"]
  C --> D["Webhook Resolves Token"]
  D --> E["System Resolves Integration"]
  E --> F["System Attributes Sale To Bound Restaurant"]
  F --> G["System Awards Loyalty"]
```

Source:
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260
samparka-backend/src/integrations/pos/controller.js:285-365

## Connect Contract

```json
{
  "integrationKey": "{{integrationKey}}",
  "restaurantId": "{{expectedRestaurantId}}",
  "restaurantName": "{{expectedRestaurantName}}"
}
```

Validation:

- `integrationKey` is required.
- `restaurantId` is required.
- `restaurantName` is optional.
- The request fails with `400` if `restaurantId` is missing.

## Webhook Contract

Send webhook events to `/webhook/restrox/{token}` with transaction data and a customer phone:

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-1001",
  "amount": 850,
  "customer": {
    "phone": "+97798XXXXXXXX"
  }
}
```

`customer.email` is optional metadata.
Payload restaurant fields such as `restaurantId`, `restaurantName`, `external_location_id`, and `external_location_name` are optional non-canonical fields for outlet-owned attribution.

Source:
samparka-backend/src/integrations/pos/partners/restrox/controller.js:9-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260

## Testing Checklist

Use [Integration Checklist](./integration-checklist) for go-live validation.

`ACTIVE` only proves the integration activated. Do not sign off until customer verification, loyalty transaction verification, and points verification are complete.

## OpenAPI

Use [openapi.yaml](./openapi.yaml) for machine-readable request and response definitions.

## Postman Collection

Use [postman-collection.json](./postman-collection.json) for hands-on testing.

## Support Contact

Use the Samparka support channel already assigned to your integration rollout.
