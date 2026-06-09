---
title: Webhook Authentication
description: Understand the canonical webhook authentication flow for RestroX event delivery into Samparka.
sidebarTitle: Webhook Auth
---

# Webhook Authentication

This page documents the canonical webhook authentication path used for RestroX event delivery.

<Note>
For the complete native authentication matrix, including partner APIs and partner customer APIs, use [Native Authentication](./native/authentication).
</Note>

## Canonical Webhook Path

RestroX webhook delivery uses token-based routing:

```text
POST /webhook/restrox/{token}
```

## What RestroX Needs To Send

RestroX does not send a separate authentication header on the canonical webhook path. The token in the URL path is the routing key Samparka uses to look up the configured location.

## What Response To Expect

If the token is valid, Samparka continues processing the request. If the token is unknown, Samparka returns:

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

## Native Integration Note

The native integration still uses per-location webhook tokens internally. The native Connection Key is not a replacement for the canonical webhook token.

## Related Documentation

- [Native Authentication](./native/authentication)
- [Connection Keys](./native/connection-keys)
- [Webhook Endpoint](./webhook-endpoint)
- [Response Reference](./response-reference)
