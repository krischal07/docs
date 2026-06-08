---
title: Authentication
description: Understand how RestroX authenticates webhook deliveries to Samparka.
sidebarTitle: Authentication
---

# Authentication

RestroX webhooks use token-based routing. Samparka provides a unique webhook token for each configured location, and RestroX includes that token in the webhook URL path.

Canonical path:

`POST /webhook/restrox/{token}`

## What RestroX Needs To Send

RestroX does not send a separate authentication header on the canonical partner path. The token in the URL is the routing key Samparka uses to look up the configured location.

## What Response To Expect

If the token is valid, Samparka continues processing the request. If the token is unknown, Samparka returns:

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

## What To Do If Something Goes Wrong

If you receive `404 Invalid webhook token`, confirm that:

1. The URL path uses the current token exactly as provided by Samparka.
2. The request is being sent to `/webhook/restrox/{token}`.
3. The webhook was configured for the intended outlet.
