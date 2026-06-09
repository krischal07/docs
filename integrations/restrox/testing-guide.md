---
title: Testing Guide
description: Validate native onboarding, customer resolution, and webhook-backed event delivery before go-live.
sidebarTitle: Testing Guide
---

# Testing Guide

This guide combines native partner onboarding checks with the existing webhook delivery tests.

See also: [Customer API](./native/customer-api), [Partner API](./native/partner-api), and [Troubleshooting](./troubleshooting).

## Native Connect Test

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

Expected result: a successful response with `RestroX connected`.

## Native Location Sync Test

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

Expected result: `Locations synchronized`. If `reviewRequired` is true, resolve those issues before proceeding.

## Customer Lookup Test

```bash
curl "https://your-domain/api/partners/restrox/customers/search?phone=9801234567" \
  -H "x-partner-key: your-partner-key" \
  -H "x-integration-key: SPK-RX-ABC12345"
```

Expected result:

- `exists: true` for an existing customer
- `exists: false` for a missing customer

## Native Test Sale

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

Expected result:

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

Verification points:

- `RawWebhookEvent` should be created for the request
- `route_token` should be populated on the raw event
- `InternalEvent` should be created when the event passes canonical ingress validation
- downstream loyalty processing should execute for processable events

Notes:

- `locationId` is the preferred provider-facing `external_location_id`
- a valid location identifier alone is insufficient; the payload must still be a valid RestroX event payload

## Direct Webhook Sale Test

```bash
curl -X POST "https://your-domain/webhook/restrox/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "restrox-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "external_location_id": "ktm-branch-01",
    "external_location_name": "Kathmandu Branch",
    "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
  }'
```

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## Duplicate Webhook Test

Resend the sale payload exactly as-is.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## Refund Test

Use a `refund.created` payload that reuses the original sale identifier.

## Invalid Token Test

Expected response:

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

## Wrong Location Test

Use a location identifier that does not resolve to a mapped location for the integration.

Expected response:

```json
{
  "success": false,
  "message": "No mapped RestroX location found for the provided location identifier"
}
```

## Related Documentation

- [Readiness Checklist](./native/readiness-checklist)
- [Webhook Endpoint](./webhook-endpoint)
- [Payload Reference](./payload-reference)
- [Customer Identity](./native/customer-identity)
