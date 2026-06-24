---
title: Quick Start
description: Connect POS to one Samparka outlet and validate partner and webhook delivery quickly.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a verified POS integration.

See also: [Testing Guide](./testing-guide) and [Integration Checklist](./integration-checklist).

<Tip>
  Use the [Postman Collection](./reference/postman-collection) if you want the easiest setup path. It already includes the partner API, webhook, and customer verification requests.
</Tip>

## 1. Receive The Integration Key

Ask Samparka to manually share both of these values for your outlet-owned POS integration:

- `integrationKey`
- provider API key to use as `Authorization: Bearer {{providerApiKey}}`

Use your provider slug in the `{provider}` route segment for all partner and webhook requests.

## 2. Connect The Location

Send `POST` requests to `/api/partners/{{provider}}/connect` with `Content-Type: application/json` and the partner auth header.

```http
Authorization: Bearer {{providerApiKey}}
Content-Type: application/json
```

Example payload:

```json
{
  "integrationKey": "{{integrationKey}}",
  "externalLocationId": "{{expectedLocationId}}",
  "externalLocationName": "{{expectedLocationName}}"
}
```

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` fields and normalizes them to `externalLocationId` and `externalLocationName`. New integrations should send the generic location fields.

Expected success response:

```json
{
  "success": true,
  "integrationId": "{{integrationId}}",
  "token": "{{webhookToken}}",
  "status": "CONNECTED"
}
```

Validate that the response:

- returns HTTP `200`
- has `"success": true`
- has `"status": "CONNECTED"`
- includes a non-empty `"integrationId"`
- includes a non-empty `"token"`

Do not assert `message`, `restaurantId`, or `externalLocationId` in the connect response. Those values are no longer part of the success payload.

## 3. Configure The Webhook URL

After a successful connect request, store the returned `token`.

Configure POS to send webhook events to:

`https://samparka.xyz/webhook/{provider}/{{webhookToken}}`

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

Location identity comes from the integration that owns `{token}`. Do not rely on payload location fields for outlet-owned attribution.

## 5. Optional Partner Shortcut

`POST /api/partners/{provider}/test-sale` submits a sale into the same webhook processing path, but wraps the webhook result:

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

POS authenticates as a partner, provides `Authorization: Bearer {{providerApiKey}}`, provides `x-integration-key`, searches the customer by phone, and receives customer loyalty data.

```bash
curl -X GET "https://your-domain/api/partners/{provider}/customers/search?phone={{customerPhone}}" \
  -H "Authorization: Bearer {{providerApiKey}}" \
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
curl -X GET "https://your-domain/api/partners/{provider}/customers/{{customerId}}" \
  -H "Authorization: Bearer {{providerApiKey}}" \
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
