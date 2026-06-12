---
title: Quick Start
description: Connect RestroX to one Samparka outlet and validate webhook delivery quickly.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a verified RestroX integration.

See also: [Testing Guide](./testing-guide) and [Integration Checklist](./integration-checklist).

## 1. Receive The Integration Key

Ask Samparka for the outlet-owned RestroX `integrationKey`.

Source:
samparka-backend/src/integrations/pos/merchant/service.js:1136-1189
samparka-backend/src/integrations/pos/partners/restrox/service.js:116-129

## 2. Connect The Restaurant

Send `POST` requests to:

`/api/partners/restrox/connect`

Use `Content-Type: application/json` and the partner auth header:

```http
x-partner-key: {{partnerKey}}
```

Payload:

```json
{
  "integrationKey": "{{integrationKey}}",
  "restaurantId": "{{expectedRestaurantId}}",
  "restaurantName": "{{expectedRestaurantName}}"
}
```

Expected response:

```json
{
  "success": true,
  "message": "RestroX connected",
  "connected": true,
  "integrationId": "{{integrationId}}",
  "externalLocationId": "{{expectedRestaurantId}}",
  "externalLocationName": "{{expectedRestaurantName}}",
  "status": "CONNECTED"
}
```

Source:
samparka-backend/src/index.js:146-155
samparka-backend/src/integrations/pos/partners/restrox/controller.js:9-28
samparka-backend/src/integrations/pos/partners/restrox/service.js:132-260

## 3. Configure The Webhook URL

After the response status is `CONNECTED`, use the integration webhook URL:

`/webhook/restrox/{token}`

Source:
samparka-backend/src/index.js:151-155
samparka-backend/src/integrations/pos/routes.js:11-15

## 4. Send A Test Sale

Use the canonical sale fixture from [`examples/payloads.json`](./examples/payloads.json):

```json
{
  "event_type": "order.completed",
  "order_id": "restrox-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": {
    "phone": "9800000101"
  },
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-30
samparka-backend/src/integrations/pos/controller.js:351-365

Restaurant identity comes from the integration that owns `{token}`. Do not rely on payload restaurant fields for outlet-owned attribution.

## 5. Verify The Integration Became ACTIVE

Fetch the merchant integration after the first valid sale and confirm the connection status is `ACTIVE`.

Minimum verification:

- `connectionStatus = ACTIVE`
- `healthStatus = HEALTHY`

`ACTIVE` confirms the integration activated, but it does not prove the loyalty workflow finished successfully.

## 6. Verify The Customer Exists

RestroX authenticates as a partner, provides `x-partner-key`, provides `x-integration-key`, searches the customer by phone, and receives customer loyalty data.

```bash
curl -X GET "https://your-domain/api/partners/restrox/customers/search?phone={{customerPhone}}" \
  -H "x-partner-key: {{partnerKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Confirm:

- the response is `200`
- a customer record exists
- the returned customer matches the phone used in the sale
- the customer belongs to the store scoped by `integrationKey`

`x-partner-key` identifies RestroX. `x-integration-key` identifies the merchant or store context used for the search.

## 7. Verify Customer Details And Points

Fetch the resolved customer:

`GET /api/customers/{customerId}`

Confirm:

- the response is `200`
- the expected customer is returned
- loyalty fields are populated
- points reflect the processed sale

## 8. Verify Loyalty Transaction Exists

Use merchant tooling or the customer details response to confirm a loyalty transaction was created for the test sale before go-live.

## 9. Repost The Same Payload Once

Send the exact same body again. Samparka should acknowledge the duplicate safely.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

Source:
samparka-backend/src/integrations/pos/controller.js:293-312

## 10. Send A Test Refund

Use a `refund.created` payload that reuses the original sale identifier.

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:27-29
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38

## 11. Complete Go-Live Validation

Run the checks in [Integration Checklist](./integration-checklist) before switching to production traffic.

Source:
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365
