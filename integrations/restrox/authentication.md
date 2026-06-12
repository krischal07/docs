---
title: Authentication
description: Understand how RestroX authenticates webhook deliveries to Samparka.
sidebarTitle: Authentication
---

# Authentication

RestroX uses partner authentication for partner API calls and token-based routing for webhooks. Samparka provides a unique webhook token for each configured location, and RestroX includes that token in the webhook URL path.

## Customer Lookup Authentication

Use the documented customer lookup contract:

```http
GET /api/partners/restrox/customers/search?phone={{customerPhone}}
x-partner-key: {{partnerKey}}
x-integration-key: {{integrationKey}}
```

Authorization model:

```txt
Partner Key
+
Integration Key
=
Customer Lookup Authorization
```

- the partner key identifies RestroX
- the integration key identifies the merchant or store context
- customer search is scoped to that integration's store

The backend also supports `Authorization: Bearer {{partnerKey}}` as an alternative to `x-partner-key`, but the documented onboarding contract uses `x-partner-key`.

Canonical path:

`POST /webhook/restrox/{token}`

Source:
samparka-backend/src/integrations/pos/routes.js:11-15
samparka-backend/src/models/posIntegrationLocationModel.js:35-40

## What RestroX Needs To Send

RestroX does not send a separate authentication header on the canonical partner path. The token in the URL is the routing key Samparka uses to look up the configured location.

Source:
samparka-backend/src/integrations/pos/controller.js:198-205
samparka-backend/src/integrations/pos/locationResolutionService.js:9-27

## What Response To Expect

If the token is valid, Samparka continues processing the request. If the token is unknown, Samparka returns:

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

Source:
samparka-backend/src/integrations/pos/controller.js:200-205

## What To Do If Something Goes Wrong

If you receive `404 Invalid webhook token`, confirm that:

1. The URL path uses the current token exactly as provided by Samparka.
2. The request is being sent to `/webhook/restrox/{token}`.
3. The webhook was configured for the intended outlet.

Source:
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/locationResolutionService.js:9-27
