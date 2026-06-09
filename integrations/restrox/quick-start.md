---
title: Quick Start
description: Connect a merchant, sync locations, verify customer behavior, and submit a test sale through the RestroX native integration.
sidebarTitle: Quick Start
---

# Quick Start

This is the fastest path to a working RestroX native integration.

See also: [Partner API](./native/partner-api), [Customer API](./native/customer-api), and [Readiness Checklist](./native/readiness-checklist).

## 1. Receive The Connection Key

Get the merchant's Samparka Connection Key from the Samparka integration setup.

Example:

```text
SPK-RX-ABC12345
```

## 2. Connect The Merchant

```bash
curl -X POST "https://your-domain/api/partners/restrox/connect" \
  -H "Content-Type: application/json" \
  -H "x-partner-key: your-partner-key" \
  --data '{
    "integrationKey": "SPK-RX-ABC12345",
    "account": {
      "id": "restrox-account-001",
      "name": "Java Express"
    },
    "locations": []
  }'
```

## 3. Sync Locations

```bash
curl -X POST "https://your-domain/api/partners/restrox/sync-locations" \
  -H "Content-Type: application/json" \
  -H "x-partner-key: your-partner-key" \
  --data '{
    "integrationKey": "SPK-RX-ABC12345",
    "account": {
      "id": "restrox-account-001",
      "name": "Java Express"
    },
    "locations": [
      {
        "externalLocationId": "ktm-branch-01",
        "externalLocationName": "Kathmandu Branch"
      }
    ]
  }'
```

If the response includes `reviewRequired: true`, resolve the location review issues before moving on.

## 4. Verify Customer Flow

Search for an existing customer using the partner customer API:

```bash
curl "https://your-domain/api/partners/restrox/customers/search?phone=9801234567" \
  -H "x-partner-key: your-partner-key" \
  -H "x-integration-key: SPK-RX-ABC12345"
```

If the customer is missing, note that customer creation is not exposed through the partner customer API.

## 5. Submit A Test Sale

```bash
curl -X POST "https://your-domain/api/partners/restrox/test-sale" \
  -H "Content-Type: application/json" \
  -H "x-partner-key: your-partner-key" \
  --data '{
    "integrationKey": "SPK-RX-ABC12345",
    "locationId": "ktm-branch-01",
    "payload": {
      "event_type": "order.completed",
      "order_id": "restrox-sale-1001",
      "created_at": "2026-06-08T10:15:00.000Z",
      "amount": 850,
      "currency": "NPR",
      "customer": { "phone": "9801234567" },
      "external_location_id": "ktm-branch-01",
      "external_location_name": "Kathmandu Branch",
      "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
    }
  }'
```

Expected response:

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

## 6. Confirm Readiness

Use the readiness checklist and merchant onboarding model to confirm the integration is ready for production.

## Webhook Transport Note

The native test-sale path resolves the mapped location, loads the stored webhook token, and enters the same canonical token-routed ingress used by production webhook traffic. If you also need to validate raw webhook delivery directly, continue with:

- [Webhook Endpoint](./webhook-endpoint)
- [Payload Reference](./payload-reference)
- [Testing Guide](./testing-guide)
