---
title: Quick Start
description: Configure RestroX webhooks with Samparka Loyalty and validate the happy path quickly.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a working RestroX integration.

See also: [Testing Guide](./testing-guide) and [Integration Checklist](./integration-checklist).

## 1. Receive Outlet Mapping Details

Ask Samparka for the webhook URL and confirm the `external_location_id` value that RestroX will send for each outlet. Samparka resolves webhook tokens to a location record and also checks the `external_location_id` in the payload.

Source:
samparka-backend/src/integrations/pos/locationResolutionService.js:9-27
samparka-backend/src/integrations/pos/locationResolutionService.js:68-77
samparka-backend/src/models/posIntegrationLocationModel.js:25-40

## 2. Configure the Webhook URL

Send `POST` requests to:

`/webhook/restrox/{token}`

Use `Content-Type: application/json`.

Source:
samparka-backend/src/index.js:151-155
samparka-backend/src/integrations/pos/routes.js:11-15
samparka-backend/src/integrations/pos/providers/restrox/validator.js:23-30

## 3. Send a Test Sale

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

## 4. Repost the Same Payload Once

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

## 5. Send a Test Refund

Use a `refund.created` payload that reuses the original sale identifier.

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:27-29
samparka-backend/src/loyalty/handlers/reversalEventHandler.js:23-38

## 6. Complete Go-Live Validation

Run the checks in [Integration Checklist](./integration-checklist) before switching to production traffic.

Source:
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365
