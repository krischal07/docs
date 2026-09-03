---
title: POS Partner Guide
description: Canonical overview of the POS partner connect, test-sale, customer lookup, and webhook integration with Samparka Loyalty.
sidebarTitle: Guide Overview
---

import { Tabs, Tab } from "@mintlify/components";

# POS Partner Guide

<Tabs>
  <Tab title="App">

Samparka is a customer loyalty platform. A valid POS integration is complete only after both transport and business outcomes are verified.

1. Connect one external location to one outlet-owned Samparka integration with `POST /api/partners/restrox/connect`.
2. Store the returned `token` and use it to configure the webhook URL.
3. Send sale, refund, and void webhook traffic after the integration is connected.
4. Verify customer resolution, loyalty processing, and awarded points after the first valid sale.

## Integration Verification Flow

1. Receive the Samparka Integration Key and provider API key manually for the outlet-owned POS integration.
2. For this integration, use `restrox` as the `{provider}` value in documented route examples.
3. Call `POST /api/partners/restrox/connect` with `integrationKey`, `externalLocationId`, and optional `externalLocationName`.
4. Store the returned `token`.
5. Configure POS to send events to `https://your-domain/webhook/restrox/{token}`.
6. Send a test `order.completed` event to the webhook endpoint or use `POST /api/partners/restrox/test-sale`.
7. Verify the integration becomes `ACTIVE`.
8. Search the customer with `GET /api/partners/restrox/customers/search?phone=...` using partner authentication and `x-integration-key`.
9. Fetch the customer with `GET /api/partners/restrox/customers/{customerId}` and confirm loyalty fields are populated.
10. Verify a loyalty transaction exists for the sale and the awarded points reflect successful processing.
11. Repost the same sale payload once to confirm duplicate safety.
12. Send a `refund.created` webhook that references the original sale identifier.
13. Complete the go-live checklist.

## Quick Links

- [Partner Handoff](./PARTNER-HANDOFF)
- [Quick Start](./quick-start)
- [Authentication](./authentication)
- [Endpoint Catalog](./endpoint-catalog)
- [Event Types](./event-types)
- [Payload Reference](./payload-reference)
- [Response Reference](./response-reference)
- [Refunds](./refunds)
- [Idempotency](./idempotency)
- [Location Binding Reference](./location-mapping)
- [Testing Guide](./testing-guide)
- [Troubleshooting](./troubleshooting)
- [Integration Checklist](./integration-checklist)
- [OpenAPI](./openapi.yaml)
- [Postman Collection](./postman-collection.json)

## Active Connect Contract

```json
{
  "integrationKey": "...",
  "externalLocationId": "...",
  "externalLocationName": "..."
}
```

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` fields and normalizes them to `externalLocationId` and `externalLocationName`. New integrations should send the generic location fields.

Successful connect responses return:

```json
{
  "success": true,
  "integrationId": "...",
  "token": "...",
  "status": "CONNECTED"
}
```

## Test Sale Contract

`POST /api/partners/restrox/test-sale` is a partner wrapper around the same webhook processing path used by `/webhook/restrox/{token}`.

Successful responses are wrapped:

```json
{
  "success": true,
  "message": "Test sale submitted",
  "data": {
    "success": true,
    "message": "Event received"
  }
}
```

Duplicate test-sale submissions return:

```json
{
  "success": true,
  "message": "Test sale submitted",
  "data": {
    "success": true,
    "message": "Event already processed"
  }
}
```

## Canonical Webhook Attribution

For outlet-owned POS, location identity is resolved from the integration binding:

```txt
Webhook Token
-> PosIntegration
-> Outlet
-> Bound External Location
```

Webhook payload location fields are optional, non-canonical metadata. They are not the source of truth for outlet-owned attribution.

## Customer Lookup Authorization

Customer lookup for POS is partner-authenticated and integration-scoped:

```txt
Provider API Key
+
Integration Key
=
Customer Lookup Authorization
```

- the provider API key is shared manually by Samparka during onboarding
- the integration key identifies the merchant or store context
- customer search is scoped to the store that owns the integration

## Important Delivery Note

A `200 Event received` response means Samparka accepted the webhook delivery. It does not guarantee that loyalty activity was created.

`ACTIVE` only proves integration activation. Business success is confirmed only after:

- the customer can be found with the sale identity
- loyalty transaction data exists for that customer
- the customer's points reflect the processed sale
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

This package is the shareable entry point for the POS connect, test-sale, customer lookup, and webhook integration with Samparka.

## Start Here

1. [Overview](./README)
2. [Quick Start](./quick-start)
3. [Endpoint Catalog](./endpoint-catalog)
4. [Payload Reference](./payload-reference)
5. [Testing Guide](./testing-guide)

## Connect Contract

Request:

```json
{
  "integrationKey": "{{integrationKey}}",
  "externalLocationId": "{{expectedLocationId}}",
  "externalLocationName": "{{expectedLocationName}}"
}
```

Success response:

```json
{
  "success": true,
  "integrationId": "{{integrationId}}",
  "token": "{{webhookToken}}",
  "status": "CONNECTED"
}
```

## Webhook Contract

Send webhook events to `/webhook/restrox/{token}` with transaction data and a customer phone:

```json
{
  "event_type": "order.completed",
  "order_id": "pos-sale-1001",
  "amount": 850,
  "customer": {
    "phone": "+97798XXXXXXXX"
  }
}
```

Payload location fields such as `external_location_id`, `external_location_name`, `restaurantId`, and `restaurantName` are optional non-canonical metadata for outlet-owned attribution.

## Customer Lookup Contract

Use the partner-authenticated customer lookup routes:

```http
GET /api/partners/restrox/customers/search?phone={{customerPhone}}
Authorization: Bearer {{providerApiKey}}
x-integration-key: {{integrationKey}}
```

```http
GET /api/partners/restrox/customers/{{customerId}}
Authorization: Bearer {{providerApiKey}}
x-integration-key: {{integrationKey}}
```

`providerApiKey` is shared manually by Samparka during onboarding. For this integration, use `restrox` as the route provider value. `x-integration-key` identifies the merchant or store context, and the search is scoped to that integration's store.

## Testing Checklist

Use [Integration Checklist](./integration-checklist) for go-live validation.

`ACTIVE` only proves the integration activated. Do not sign off until customer verification, loyalty transaction verification, and points verification are complete.

## OpenAPI

Use [openapi.yaml](./openapi.yaml) for machine-readable request and response definitions.

## Postman Collection

Use [postman-collection.json](./postman-collection.json) for hands-on testing.

## Support Contact

Use the Samparka support channel already assigned to your integration rollout.
  </Tab>
</Tabs>
