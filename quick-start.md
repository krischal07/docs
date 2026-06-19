---
title: Quick Start
description: Connect POS to one Samparka outlet and validate partner and webhook delivery quickly.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a verified POS integration.

See also: [Testing Guide](./testing-guide) and [Integration Checklist](./integration-checklist).

## 1. Receive The Integration Key

Ask Samparka to manually share both of these values for your outlet-owned POS integration:

- `integrationKey`
- partner API key to use as `x-partner-key`

For this integration, replace `{provider}` with `restrox`.

## 2. Connect The Restaurant

Send `POST` requests to `/api/partners/{{provider}}/connect` with `Content-Type: application/json` and the partner auth header.

```http
x-partner-key: {{partnerKey}}
```

Example payload:

```json
{
  "integrationKey": "{{integrationKey}}",
  "restaurantId": "{{expectedRestaurantId}}",
  "restaurantName": "{{expectedRestaurantName}}"
}
```

Expected success response:

```json
{
  "success": true,
  "integrationId": "{{integrationId}}",
  "token": "{{token}}",
  "status": "CONNECTED"
}
```

## 3. Configure The Webhook URL

After a successful connect request, store the returned `token`.

Configure POS to send webhook events to:

`https://samparka.xyz/webhook/restrox/{{token}}`

## 4. Send A Test Sale

Use the canonical sale fixture from [`examples/payloads.json`](./examples/payloads.json):

```json
{
  "event_type": "order.completed",
  "order_id": "pos-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": {
    "phone": "9800000101"
  },
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

Expected webhook response:

```json
{
  "success": true,
  "message": "Event received"
}
```

Restaurant identity comes from the integration that owns `{token}`. Do not rely on payload restaurant fields for outlet-owned attribution.

## 5. Optional Partner Shortcut

`POST /api/partners/restrox/test-sale` submits a sale into the same webhook processing path, but wraps the webhook result:

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

## 6. Verify The Integration Became ACTIVE

Fetch the merchant integration after the first valid sale and confirm:

- `connectionStatus = ACTIVE`
- `healthStatus = HEALTHY`

`ACTIVE` confirms the integration activated, but it does not prove the loyalty workflow finished successfully.

## 7. Verify The Customer Exists

POS authenticates as a partner, provides `x-partner-key`, provides `x-integration-key`, searches the customer by phone, and receives customer loyalty data.

```bash
curl -X GET "https://your-domain/api/partners/restrox/customers/search?phone={{customerPhone}}" \
  -H "x-partner-key: {{partnerKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected hit response:

```json
{
  "exists": true,
  "customer": {
    "id": "{{customerId}}",
    "phone": "{{customerPhone}}",
    "points": 85
  }
}
```

Expected miss response:

```json
{
  "exists": false
}
```

## 8. Verify Customer Details And Points

Fetch the resolved customer:

```bash
curl -X GET "https://your-domain/api/partners/restrox/customers/{{customerId}}" \
  -H "x-partner-key: {{partnerKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected response:

```json
{
  "customer": {
    "id": "{{customerId}}",
    "phone": "{{customerPhone}}",
    "points": 85,
    "lifetimePoints": 85
  }
}
```

## 9. Verify Loyalty Transaction Exists

Use merchant tooling or the verified customer state to confirm a loyalty transaction was created for the test sale before go-live.

## 10. Repost The Same Payload Once

Send the exact same webhook body again. Samparka should acknowledge the duplicate safely.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## 11. Send A Test Refund

Use a `refund.created` payload that reuses the original sale identifier.

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## 12. Complete Go-Live Validation

Run the checks in [Integration Checklist](./integration-checklist) before switching to production traffic.
