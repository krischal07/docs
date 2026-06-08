---
title: Webhook Endpoint
description: Configure the RestroX webhook endpoint, request shape, and acknowledgment behavior.
sidebarTitle: Webhook Endpoint
---

# Webhook Endpoint

## Endpoint

- Method: `POST`
- Path: `/webhook/restrox/{token}`
- Content-Type: `application/json`

Source:
samparka-backend/src/index.js:151-155
samparka-backend/src/integrations/pos/routes.js:11-15
samparka-backend/src/integrations/pos/providers/restrox/validator.js:23-30

## What RestroX Needs To Send

Send a JSON object body with an `event_type` and the fields described in [Payload Reference](./payload-reference).

Source:
samparka-backend/src/integrations/pos/providers/restrox/parser.js:18-61

## What Response To Expect

Samparka returns JSON acknowledgments:

- `200 Event received`
- `200 Event already processed`
- `400 Request body must be a JSON object`
- `400 Missing event_type in payload`
- `404 Invalid webhook token`
- `500 Internal server error`

Source:
samparka-backend/src/integrations/pos/providers/restrox/validator.js:23-26
samparka-backend/src/integrations/pos/providers/restrox/parser.js:20-23
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365
samparka-backend/src/integrations/pos/controller.js:397-397

## Acknowledgment Timing

Samparka stores and validates the incoming request before returning the final acknowledgment for the canonical webhook path. A successful acknowledgment means the delivery was accepted. It does not guarantee that loyalty activity was created.

Source:
samparka-backend/src/integrations/pos/controller.js:181-243
samparka-backend/src/integrations/pos/controller.js:332-365
samparka-backend/src/loyalty/handlers/saleCompletedHandler.js:101-110
samparka-backend/src/loyalty/handlers/saleCompletedHandler.js:134-142

## What To Do If Something Goes Wrong

Use [Response Reference](./response-reference) and [Troubleshooting](./troubleshooting) to decide whether to retry, correct the payload, or confirm outlet mapping with Samparka.
