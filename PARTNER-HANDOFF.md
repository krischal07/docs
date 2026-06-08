---
title: RestroX Partner Handoff
description: Canonical partner handoff summary for RestroX onboarding with Samparka Loyalty.
sidebarTitle: Partner Handoff Source
---

# RestroX Partner Handoff

This package is the shareable entry point for RestroX webhook integration with Samparka. Samparka accepts sale, refund, and void webhook events so restaurants can connect RestroX transactions to loyalty processing. The backend exposes a token-based webhook URL, expects JSON payloads, and returns a simple acknowledgment response. Integration effort is low if your team already sends webhook events from RestroX.

Source:
samparka-backend/src/index.js:145-155
samparka-backend/src/integrations/pos/routes.js:11-15
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-30

## Start Here

1. [Overview](./README)
2. [Quick Start](./quick-start)
3. [Webhook Endpoint](./webhook-endpoint)
4. [Payload Reference](./payload-reference)
5. [Testing Guide](./testing-guide)

## Integration Flow

```mermaid
flowchart TD
  A["Receive webhook token"] --> B["Configure RestroX webhook"]
  B --> C["Send test sale"]
  C --> D["Verify acknowledgment"]
  D --> E["Test duplicate repost"]
  E --> F["Test refund"]
  F --> G["Production signoff"]
```

Source:
samparka-backend/src/integrations/pos/routes.js:11-15
samparka-backend/src/integrations/pos/controller.js:200-205
samparka-backend/src/integrations/pos/controller.js:301-365

## Supported Events

- `order.completed`
- `refund.created`
- `order.voided`

Source:
samparka-backend/src/integrations/pos/providers/restrox/mapper.js:18-30

## Testing Checklist

Use [Integration Checklist](./integration-checklist) for go-live validation.

## OpenAPI

Use [openapi.yaml](./openapi.yaml) for machine-readable request and response definitions.

## Postman Collection

Use [postman-collection.json](./postman-collection.json) for hands-on testing.

## Support Contact

Implementation Detail Requires Confirmation
