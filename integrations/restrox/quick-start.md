---
title: Quick Start
description: Connect RestroX to one Samparka outlet and validate webhook delivery quickly.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a working RestroX integration.

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
  "customer": { "phone": "9800000101" },
  "external_location_id": "ktm-branch-01",
  "external_location_name": "Kathmandu Branch",
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

## 5. Repost The Same Payload Once

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

## 6. Send A Test Refund

Use a `refund.created` payload that reuses the original sale identifier.

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:27-29
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38

## 7. Complete Go-Live Validation

Run the checks in [Integration Checklist](./integration-checklist) before switching to production traffic.

Source:
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365
