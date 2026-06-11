---
title: RestroX Partner Guide
description: Canonical overview of the outlet-owned RestroX integration with Samparka.
sidebarTitle: Guide Overview
---

# RestroX Partner Guide

RestroX integrates with Samparka as an outlet-owned provider. One outlet owns one RestroX integration, one Integration Key, one webhook token, and one restaurant binding.

## Core Rules

1. `provider = restrox`
2. `ownership_mode = OUTLET_OWNED`
3. Creation requires `storeId` and `outletId`
4. Restaurant binding is stored directly on `PosIntegration`
5. Webhooks route with `POST /webhook/restrox/{token}`

## Lifecycle

Integration lifecycle:

```text
CREATED -> CONNECTED -> ACTIVE -> DISCONNECTED
```

Health is tracked separately:

```text
HEALTHY | WARNING | ERROR
```

## Activation Rule

RestroX becomes `ACTIVE` after:

1. the integration is connected to a restaurant
2. the first valid `sale.completed` event is received

## Not Used By RestroX

RestroX does not rely on store-owned location workflows. Compatibility pages are kept only so existing URLs remain valid.

## Quick Links

- [Architecture](./native/architecture)
- [Partner API](./native/partner-api)
- [Merchant Onboarding](./native/merchant-onboarding)
- [Webhook Endpoint](./webhook-endpoint)
- [Payload Reference](./payload-reference)
- [Testing Guide](./testing-guide)
- [Reference Assets](./reference/overview)
