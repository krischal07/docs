---
title: RestroX Partner Guide
description: Canonical overview of the RestroX partner connect, test-sale, customer lookup, and webhook integration with Samparka Loyalty.
sidebarTitle: Guide Overview
---

# RestroX Partner Guide

Samparka is a customer loyalty platform. A valid RestroX integration is complete only after both transport and business outcomes are verified.

1. Connect one RestroX restaurant to one outlet-owned Samparka integration with `POST /api/partners/restrox/connect`.
2. Store the returned `token` and use it to configure the webhook URL.
3. Send sale, refund, and void webhook traffic after the integration is connected.
4. Verify customer resolution, loyalty processing, and awarded points after the first valid sale.

## Integration Verification Flow

1. Receive the Samparka Integration Key for the outlet-owned RestroX integration.
2. Call `POST /api/partners/restrox/connect` with `integrationKey`, `restaurantId`, and optional `restaurantName`.
3. Store the returned `token`.
4. Configure RestroX to send events to `https://your-domain/webhook/restrox/{token}`.
5. Send a test `order.completed` event to the webhook endpoint or use `POST /api/partners/restrox/test-sale`.
6. Verify the integration becomes `ACTIVE`.
7. Search the customer with `GET /api/partners/restrox/customers/search?phone=...` using partner authentication and `x-integration-key`.
8. Fetch the customer with `GET /api/partners/restrox/customers/{customerId}` and confirm loyalty fields are populated.
9. Verify a loyalty transaction exists for the sale and the awarded points reflect successful processing.
10. Repost the same sale payload once to confirm duplicate safety.
11. Send a `refund.created` webhook that references the original sale identifier.
12. Complete the go-live checklist.

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
- [Restaurant Binding Reference](./location-mapping)
- [Testing Guide](./testing-guide)
- [Troubleshooting](./troubleshooting)
- [Integration Checklist](./integration-checklist)
- [OpenAPI](./openapi.yaml)
- [Postman Collection](./postman-collection.json)

## Active Connect Contract

```json
{
  "integrationKey": "...",
  "restaurantId": "...",
  "restaurantName": "..."
}
```

Successful connect responses return:

```json
{
  "success": true,
  "message": "RestroX connected",
  "connected": true,
  "integrationId": "...",
  "token": "...",
  "restaurantId": "...",
  "externalLocationId": "...",
  "externalLocationName": "...",
  "status": "CONNECTED"
}
```

If the same restaurant is already bound, the response may also include `idempotent: true`.

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

For outlet-owned RestroX, restaurant identity is resolved from the integration binding:

```txt
Webhook Token
-> PosIntegration
-> Outlet
-> Bound Restaurant
```

Webhook payload restaurant fields are optional, non-canonical metadata. They are not the source of truth for restaurant attribution.

## Customer Lookup Authorization

Customer lookup for RestroX is partner-authenticated and integration-scoped:

```txt
Partner Key
+
Integration Key
=
Customer Lookup Authorization
```

- the partner key identifies RestroX
- the integration key identifies the merchant or store context
- customer search is scoped to the store that owns the integration

## Important Delivery Note

A `200 Event received` response means Samparka accepted the webhook delivery. It does not guarantee that loyalty activity was created.

`ACTIVE` only proves integration activation. Business success is confirmed only after:

- the customer can be found with the sale identity
- loyalty transaction data exists for that customer
- the customer's points reflect the processed sale
