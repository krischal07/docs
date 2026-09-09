# System Partner Guide — Complete Integration Documentation

> One-file reference for connecting a System (POS) outlet to Samparka Loyalty: connect, test-sale, customer lookup, webhooks, refunds, idempotency, troubleshooting, and the Purchase QR checkout flow.

Samparka is a customer loyalty platform. A valid System integration is complete only after **both transport and business outcomes are verified**:

1. Connect one external location to one outlet-owned Samparka integration with `POST /api/partners/{provider}/connect`.
2. Store the returned `token` and use it to configure the webhook URL.
3. Send sale, refund, and void webhook traffic after the integration is connected.
4. Verify customer resolution, loyalty processing, and awarded points after the first valid sale.

**Provider value:** for this integration, use your assigned provider slug as the `{provider}` value in documented route examples.

---

## Table of Contents

1. [Integration Verification Flow](#integration-verification-flow)
2. [Authentication](#authentication)
3. [Endpoint Catalog](#endpoint-catalog)
4. [Event Types](#event-types)
5. [Quick Start](#quick-start)
6. [Connect Contract](#connect-contract)
7. [Test Sale Contract](#test-sale-contract)
8. [Webhook Contract](#webhook-contract)
9. [Customer Lookup](#customer-lookup)
10. [Payload Reference](#payload-reference)
11. [Response Reference](#response-reference)
12. [Refunds](#refunds)
13. [Idempotency](#idempotency)
14. [Location Binding Reference](#location-binding-reference)
15. [Testing Guide](#testing-guide)
16. [Troubleshooting](#troubleshooting)
17. [Integration Checklist](#integration-checklist)
18. [Examples](#examples)
19. [Purchase QR Checkout (Communication)](#purchase-qr-checkout-communication)
20. [Known Behaviors and Limitations](#known-behaviors-and-limitations)
21. [Support Contact](#support-contact)

---

## Integration Verification Flow

```txt
Create Integration
↓
Connect Location
↓
Store Returned Token
↓
Send Sale Or Submit Test Sale
↓
Verify Integration Became ACTIVE
↓
Verify Customer Exists
↓
Verify Loyalty Transaction Exists
↓
Verify Points Awarded
```

1. Receive the Samparka **Integration Key** and **provider API key** manually for the outlet-owned System integration.
2. Call `POST /api/partners/{provider}/connect` with `integrationKey`, `externalLocationId`, and optional `externalLocationName`.
3. Store the returned `token`.
4. Configure System to send events to `https://samparka.co/webhook/{provider}/{token}`.
5. Send a test `order.completed` event to the webhook endpoint or use `POST /api/partners/{provider}/test-sale`.
6. Verify the integration becomes `ACTIVE`.
7. Search the customer with `GET /api/partners/{provider}/customers/search?phone=...` using partner authentication and `x-integration-key`.
8. Fetch the customer with `GET /api/partners/{provider}/customers/{customerId}` and confirm loyalty fields are populated.
9. Verify a loyalty transaction exists for the sale and the awarded points reflect successful processing.
10. Repost the same sale payload once to confirm duplicate safety.
11. Send a `refund.created` webhook that references the original sale identifier.
12. Complete the go-live checklist.

### End-To-End Flow

```txt
Merchant Creates Integration
        │
        ▼
Partner Connects Location
        │
        ▼
Connect Returns Token
        │
        ▼
System Sends Webhook Or Test Sale
        │
        ▼
System Resolves Integration
        │
        ▼
System Attributes Sale To Bound Location
        │
        ▼
System Calls Partner Customer API
        │
        ▼
System Returns Store-Scoped Loyalty Data
```

### Important Delivery Note

- A `200 Event received` response means Samparka **accepted the webhook delivery**. It does **not** guarantee that loyalty activity was created.
- `ACTIVE` only proves integration activation. Business success is confirmed only after:
  - the customer can be found with the sale identity
  - loyalty transaction data exists for that customer
  - the customer's points reflect the processed sale

---

## Authentication

- Merchant lifecycle APIs use merchant auth.
- Partner APIs use `Authorization: Bearer {{providerApiKey}}` as the **canonical auth model**.
- For this integration, use your assigned provider slug as the `{provider}` value in documented partner and webhook routes.
- System customer APIs use `Authorization: Bearer {{providerApiKey}}` **plus** `x-integration-key`.
- `x-partner-key` remains legacy compatibility behavior and should **not** be used in new integrations.
- Webhooks use `POST /webhook/{provider}/{token}` — the token in the URL path is the auth.

**Customer Lookup Authorization:**

```txt
Provider API Key
+
Integration Key
=
Customer Lookup Authorization
```

- The provider API key is shared manually by Samparka during onboarding.
- The integration key identifies the merchant or store context.
- Customer search is scoped to the store that owns the integration.

---

## Endpoint Catalog

| Endpoint | Purpose | Authentication |
| -------- | ------- | -------------- |
| `POST /api/partners/{provider}/connect` | Bind one external location to one outlet-owned Samparka integration. | `Authorization: Bearer {{providerApiKey}}` |
| `POST /api/partners/{provider}/test-sale` | Submit a partner-side test sale into the webhook processing path. | `Authorization: Bearer {{providerApiKey}}` |
| `GET /api/partners/{provider}/customers/search` | Search for one customer by phone within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
| `GET /api/partners/{provider}/customers/{customerId}` | Fetch one customer record within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
| `POST /integrations/system/{provider}/purchase-qr` | Create a scannable purchase-checkout QR for a cash sale from the System terminal; returns the QR link in the response. | `Authorization: Bearer {{providerApiKey}}` (store resolved from API key scope) |

---

## Event Types

Samparka's recommended System event types:

- `order.completed` — completed sales
- `refund.created` — refunds
- `order.voided` — voided or canceled sales

These map to provider-neutral events `sale.completed`, `refund.created`, and `sale.voided` internally.

---

## Quick Start

This is the fastest path to a verified System integration.

> **Tip:** Use the Postman Collection if you want the easiest setup path — it already includes the partner API, webhook, and customer verification requests.

### 1. Receive The Integration Key

Ask Samparka to manually share both of these values for your outlet-owned System integration:

- `integrationKey`
- provider API key to use as `Authorization: Bearer {{providerApiKey}}`

### 2. Connect The Location

Send `POST` requests to `/api/partners/{provider}/connect` with `Content-Type: application/json` and the partner auth header.

```http
Authorization: Bearer {{providerApiKey}}
Content-Type: application/json
```

Example payload:

```json
{
  "integrationKey": "{{integrationKey}}",
  "externalLocationId": "{{expectedLocationId}}",
  "externalLocationName": "{{expectedLocationName}}"
}
```

Expected success response:

```json
{
  "success": true,
  "integrationId": "{{integrationId}}",
  "token": "{{webhookToken}}",
  "status": "CONNECTED"
}
```

Validate that the response:

- returns HTTP `200`
- has `"success": true`
- has `"status": "CONNECTED"`
- includes a non-empty `"integrationId"`
- includes a non-empty `"token"`

Do **not** assert `message`, `restaurantId`, or `externalLocationId` in the connect response. Those values are no longer part of the success payload.

### 3. Configure The Webhook URL

After a successful connect request, store the returned `token` and configure System to send webhook events to:

```
https://samparka.co/webhook/{provider}/{webhookToken}
```

### 4. Send A Test Sale

Use the canonical sale fixture:

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": {
    "phone": "9800000101"
  },
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

Expected webhook response:

```json
{
  "success": true,
  "message": "Event received"
}
```

Location identity comes from the integration that owns `{token}`. Do **not** rely on payload location fields for outlet-owned attribution.

### 5. Optional Partner Shortcut

`POST /api/partners/{provider}/test-sale` submits a sale into the same webhook processing path, but wraps the webhook result:

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

### 6. Verify The Integration Became ACTIVE

Fetch the merchant integration after the first valid sale and confirm:

- `connectionStatus = ACTIVE`
- `healthStatus = HEALTHY`

`ACTIVE` confirms the integration activated, but it does **not** prove the loyalty workflow finished successfully.

### 7. Verify The Customer Exists

```bash
curl -X GET "https://samparka.co/api/partners/{provider}/customers/search?phone={{customerPhone}}" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected hit response:

```json
{
  "exists": true,
  "customer": {
    "id": "{{customerId}}",
    "phone": "{{customerPhone}}",
    "points": 85
  }
}
```

Expected miss response:

```json
{
  "exists": false
}
```

### 8. Verify Customer Details And Points

```bash
curl -X GET "https://samparka.co/api/partners/{provider}/customers/{{customerId}}" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected response:

```json
{
  "customer": {
    "id": "{{customerId}}",
    "phone": "{{customerPhone}}",
    "points": 85,
    "lifetimePoints": 85
  }
}
```

### 9. Verify Loyalty Transaction Exists

Use merchant tooling or the verified customer state to confirm a loyalty transaction was created for the test sale before go-live.

### 10. Repost The Same Payload Once

Send the exact same webhook body again. Samparka should acknowledge the duplicate safely:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

### 11. Send A Test Refund

Use a `refund.created` payload that reuses the original sale identifier:

```json
{
  "success": true,
  "message": "Event received"
}
```

### 12. Complete Go-Live Validation

Run the checks in the [Integration Checklist](#integration-checklist) before switching to production traffic.

---

## Connect Contract

`POST /api/partners/{provider}/connect`

Request:

```json
{
  "integrationKey": "{{integrationKey}}",
  "externalLocationId": "{{expectedLocationId}}",
  "externalLocationName": "{{expectedLocationName}}"
}
```

Success response:

```json
{
  "success": true,
  "integrationId": "{{integrationId}}",
  "token": "{{webhookToken}}",
  "status": "CONNECTED"
}
```

Validation:

- `integrationKey` is **required**.
- `externalLocationId` is **required**.
- `externalLocationName` is optional.

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` fields and normalizes them to `externalLocationId` and `externalLocationName`. **New integrations should send the generic location fields.**

---

## Test Sale Contract

`POST /api/partners/{provider}/test-sale` is a partner wrapper around the same webhook processing path used by `/webhook/{provider}/{token}`.

Request:

```json
{
  "integrationKey": "system-key-001",
  "payload": {
    "event_type": "order.completed",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": {
      "phone": "9800000101"
    },
    "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
  }
}
```

Successful responses are wrapped:

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

Duplicate test-sale submissions return:

```json
{
  "success": true,
  "message": "Test sale submitted",
  "data": {
    "success": true,
    "message": "Event already processed"
  }
}
```

---

## Webhook Contract

Send webhook events to `/webhook/{provider}/{token}` with transaction data and a customer phone:

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-1001",
  "amount": 850,
  "customer": {
    "phone": "+97798XXXXXXXX"
  }
}
```

Payload location fields such as `external_location_id`, `external_location_name`, `restaurantId`, and `restaurantName` are **optional non-canonical metadata** for outlet-owned attribution.

### Canonical Webhook Attribution

For outlet-owned System, location identity is resolved from the integration binding:

```txt
Webhook Token
-> SystemIntegration
-> Outlet
-> Bound External Location
```

Webhook payload location fields are optional, non-canonical metadata. They are **not** the source of truth for outlet-owned attribution.

---

## Customer Lookup

Use the partner-authenticated customer lookup routes:

```http
GET /api/partners/{provider}/customers/search?phone={{customerPhone}}
Authorization: Bearer {{providerApiKey}}
x-integration-key: {{integrationKey}}
```

```http
GET /api/partners/{provider}/customers/{{customerId}}
Authorization: Bearer {{providerApiKey}}
x-integration-key: {{integrationKey}}
```

`providerApiKey` is shared manually by Samparka during onboarding. Use your assigned provider slug as the route provider value. `x-integration-key` identifies the merchant or store context, and the search is scoped to that integration's store.

---

## Payload Reference

Samparka reads different fields for `connect`, `test-sale`, and direct webhook delivery.

### Connect Request — `POST /api/partners/{provider}/connect`

**Required Properties**

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `SystemIntegration`. | `system-key-001` |
| `externalLocationId` | string | Yes | System location identifier persisted as `external_location_id`. (Legacy alias: `restaurantId`.) | `ktm-branch-01` |

**Optional Properties**

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `externalLocationName` | string | No | Human-readable location name persisted as `external_location_name`. (Legacy alias: `restaurantName`.) | `Kathmandu Branch` |

### Test Sale Request — `POST /api/partners/{provider}/test-sale`

**Required Properties**

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `integrationKey` | string | Yes | Samparka Integration Key that resolves one outlet-owned `SystemIntegration`. | `system-key-001` |
| `payload` | object | Yes | Webhook-shaped event body submitted into the webhook processing pipeline. | See webhook fields below. |

### Webhook Fields — `POST /webhook/{provider}/{token}`

**Required Fields**

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `event_type` or `type` | string | Yes | System event name. | `order.completed` |

**Optional Fields**

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `order_id` or `transaction_id` or `id` | string | No | Provider transaction identifier used for transaction reference and idempotency. | `system-sale-1001` |
| `created_at` or `timestamp` | string | No | Event time from System. | `2026-06-08T10:15:00.000Z` |
| `amount` or `order_total` or `total` | number | No | Transaction amount. | `850` |
| `currency` or `currency_code` | string | No | Currency code. Defaults to `NPR` if omitted. | `NPR` |
| `customer.phone` or `phone` or `customer_phone` | string | No | Primary customer identifier for search and loyalty attribution. | `9800000101` |
| `items` or `line_items` | array | No | Line items for the sale or refund. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |
| `restaurantId` or `restaurant_id` or `external_location_id` or `location_id` or `outlet_id` or `branch_id` | string | No | Optional non-canonical restaurant metadata. Outlet-owned attribution is resolved from the integration binding instead. | `ktm-branch-01` |
| `restaurantName` or `restaurant_name` or `external_location_name` or `location_name` or `branch_name` or `outlet_name` | string | No | Optional non-canonical restaurant label. | `Kathmandu Branch` |

### Restaurant Attribution Source Of Truth

For outlet-owned System:

```txt
Webhook Token
-> SystemIntegration
-> Outlet
-> Bound Restaurant
```

Restaurant identity is resolved from `integration.external_location_id` and `integration.external_location_name`, **not** from webhook payload fields.

### Additional Notes

- Webhook payload restaurant fields are optional, non-canonical metadata for outlet-owned integrations.
- Fields outside the parser mappings are not required for the canonical partner flow.

---

## Response Reference

These are the partner-visible responses for the active System integration path.

### Connect Success

```json
{
  "success": true,
  "integrationId": "684915c401ec340433d84ee2",
  "token": "abcxyz",
  "status": "CONNECTED"
}
```

### Test Sale Success

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

### Test Sale Duplicate

```json
{
  "success": true,
  "message": "Test sale submitted",
  "data": {
    "success": true,
    "message": "Event already processed"
  }
}
```

### Customer Search Hit

```json
{
  "exists": true,
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "system-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

### Customer Search Miss

```json
{
  "exists": false
}
```

### Customer Detail

```json
{
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "system-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

### Webhook And Partner Error Envelopes

| Status | Body |
| ------ | ---- |
| `200 Event received` | `{ "success": true, "message": "Event received" }` |
| `200 Event already processed` | `{ "success": true, "message": "Event already processed" }` |
| `400 Request body must be a JSON object` | `{ "success": false, "message": "Request body must be a JSON object" }` |
| `400 Missing event_type in payload` | `{ "success": false, "message": "Missing event_type in payload" }` |
| `404 Invalid webhook token` | `{ "success": false, "message": "Invalid webhook token" }` |
| `409 Integration disconnected` | `{ "success": false, "message": "Integration disconnected" }` |
| `409 Integration is not connected to a restaurant` | `{ "success": false, "message": "Integration is not connected to a restaurant" }` |
| `409 Integration is not ready to receive webhooks` | `{ "success": false, "message": "Integration is not ready to receive webhooks" }` |

### Partner Customer Error Envelope

```json
{
  "error": "partner_customer_error",
  "message": "Phone is required"
}
```

The same `{ error, message }` envelope is used for partner customer validation, auth, and lookup failures.

---

## Refunds

Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should **reference the original sale identifier** that was already sent to Samparka.

### Refund Webhook Example

```bash
curl -X POST "https://samparka.co/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "refund.created",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T11:30:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": {
      "phone": "9800000101"
    },
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

---

## Idempotency

Duplicate System deliveries return `200 Event already processed` instead of creating duplicate downstream activity. If System sends the same webhook more than once, Samparka handles it safely.

The durable dedup record is a unique index on `(provider, system_integration_id, idempotency_key)`, where the key is the event type plus provider ID, or a SHA-256 fallback of selected transaction fields.

**Treat `Event already processed` as success and stop retrying that payload.**

---

## Location Binding Reference

System connect is outlet-owned and singular:

```txt
One Integration
=
One Outlet
-> One External Location
```

The `connect` step persists:

- `external_location_id`
- `external_location_name`

Webhook delivery then resolves the bound location from the integration that owns the webhook token.

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` on connect and normalizes them to the location fields above. **New integrations should send `externalLocationId` and `externalLocationName`.**

---

## Testing Guide

This guide uses the shared fixtures in `examples/payloads.json`.

### Connect Test

```bash
curl -X POST "https://samparka.co/api/partners/{provider}/connect" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "externalLocationId": "{{expectedLocationId}}",
    "externalLocationName": "{{expectedLocationName}}"
  }'
```

Expected success response:

```json
{
  "success": true,
  "integrationId": "replace-with-real-id",
  "token": "replace-with-real-token",
  "status": "CONNECTED"
}
```

Validate that the response:

- returns HTTP `200`
- has `"success": true`
- has `"status": "CONNECTED"`
- includes a non-empty `"integrationId"`
- includes a non-empty `"token"`

Do not assert `message`, `restaurantId`, or `externalLocationId` in the connect response.

### Test Sale Wrapper Check

```bash
curl -X POST "https://samparka.co/api/partners/{provider}/test-sale" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  --data '{
    "integrationKey": "{{integrationKey}}",
    "payload": {
      "event_type": "order.completed",
      "order_id": "system-sale-1001",
      "created_at": "2026-06-08T10:15:00.000Z",
      "amount": 850,
      "currency": "NPR",
      "customer": {
        "phone": "9800000101"
      },
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

### Direct Webhook Sale Test

```bash
curl -X POST "https://samparka.co/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": {
      "phone": "9800000101"
    },
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

### Customer Search After First Sale

```bash
curl -X GET "https://samparka.co/api/partners/{provider}/customers/search?phone=9800000101" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected hit response:

```json
{
  "exists": true,
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "system-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

Expected miss response:

```json
{
  "exists": false
}
```

### Customer Detail Verification

```bash
curl -X GET "https://samparka.co/api/partners/{provider}/customers/{customerId}" \
  -H "Authorization: Bearer {{providerApiKey}}" \
  -H "x-integration-key: {{integrationKey}}"
```

Expected response:

```json
{
  "customer": {
    "id": "684a00000000000000001001",
    "phone": "9800000101",
    "points": 85,
    "lifetimePoints": 85
  }
}
```

### Duplicate Webhook Test

Resend the sale test payload exactly as-is. Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

### Invalid Token Test

```bash
curl -X POST "https://samparka.co/webhook/{provider}/{invalid-token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "system-sale-1001",
    "created_at": "2026-06-08T10:15:00.000Z",
    "amount": 850,
    "currency": "NPR",
    "customer": { "phone": "9800000101" },
    "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
  }'
```

Expected response:

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

### Missing Location Binding Test

Use the token from a newly created integration before calling connect:

```bash
curl -X POST "https://samparka.co/webhook/{provider}/{token}" \
  -H "Content-Type: application/json" \
  --data '{
    "event_type": "order.completed",
    "order_id": "system-sale-2001",
    "amount": 600,
    "currency": "NPR",
    "customer": { "phone": "9800000103" }
  }'
```

Expected response:

```json
{
  "success": false,
  "message": "Integration is not connected to a restaurant"
}
```

### Disconnected Integration Test

After disconnecting the integration, submit the same webhook again. Expected response:

```json
{
  "success": false,
  "message": "Integration disconnected"
}
```

### Validation Failure Tests

Non-object body response:

```json
{
  "success": false,
  "message": "Request body must be a JSON object"
}
```

Missing event type response:

```json
{
  "success": false,
  "message": "Missing event_type in payload"
}
```

---

## Troubleshooting

| Problem | Likely Cause | Verification Steps | Resolution |
| ------- | ------------ | ------------------ | ---------- |
| `400 externalLocationId is required` on connect | The request omitted the preferred location field or still relies on stale request examples. | Inspect the JSON body for top-level `externalLocationId`. | Send the connect payload with `integrationKey` and `externalLocationId`. |
| `404 Invalid Integration Key` on connect | The `integrationKey` does not resolve a System integration. | Compare the request key with the current Samparka Integration Key. | Replace the key and retry the connect request. |
| `409 Disconnect the integration before rebinding it to another restaurant` | The integration is already connected to a different location. | Compare the requested `externalLocationId` with the current binding. | Disconnect first, then reconnect with the new location. |
| `404 Invalid webhook token` | Wrong token in the URL path. | Compare the configured URL with the token provided by Samparka. | Update the webhook URL and retry. |
| `400 Request body must be a JSON object` | The request body is missing, malformed, or not a JSON object. | Check `Content-Type`, body encoding, and confirm the top-level payload is an object. | Correct the body and resend. |
| `400 Missing event_type in payload` | `event_type` and `type` are both missing. | Inspect the JSON body and confirm that one event type field is present. | Add `event_type` and resend. |
| `200 Event already processed` | The same payload was already delivered. | Compare the repeated request body with the original request. | Treat the duplicate as success and stop retrying it. |
| `200 Event received` but expected loyalty activity is missing | The integration is disconnected, missing its bound location, the customer phone is missing, or refund linkage needs review. | Confirm the integration is still connected, the bound location exists on the integration, the customer phone is present, and the original sale identifier is correct. | Reconnect or rebind the integration if needed, then resend a valid event. |
| `500 Internal server error` | Unexpected server-side failure. | Wait and retry the same payload later. | Retry later. If the issue repeats, contact Samparka. |

---

## Integration Checklist

Use this final go-live checklist for validating a System to Samparka integration.

### Connect Contract

- [ ] `POST /api/partners/{provider}/connect` tested
- [ ] `integrationKey` verified
- [ ] `externalLocationId` verified
- [ ] Bound external location stored on the integration
- [ ] Response status is `CONNECTED`
- [ ] Only the location-based connect payload is used in active tooling and tests

### Business Verification

- [ ] Integration became `ACTIVE` after first valid sale
- [ ] Customer found via `GET /api/partners/{provider}/customers/search?phone=...`
- [ ] Customer detail shows populated loyalty fields (points, lifetimePoints)
- [ ] Loyalty transaction exists for the sale
- [ ] Points reflect the processed sale
- [ ] Same sale payload reposted once → `200 Event already processed`
- [ ] `refund.created` referencing the original sale identifier → `200 Event received`

---

## Examples

### 1. Sale Completed

**Request:**

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": { "phone": "9800000101" },
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Event received"
}
```

**What Happened:** Samparka accepted the sale webhook, resolved the integration from the webhook token, and continued processing it as a completed sale event.

**What To Do Next:** Verify the integration becomes `ACTIVE`, then repeat the same payload once to confirm duplicate handling.

### 2. Refund Created

**Request:**

```json
{
  "event_type": "refund.created",
  "order_id": "system-sale-1001",
  "created_at": "2026-06-08T11:30:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": { "phone": "9800000101" },
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Event received"
}
```

**What Happened:** Samparka accepted the refund webhook and used the sale identifier to match the original sale.

**What To Do Next:** If you retry refund deliveries, resend the exact same payload once and confirm the duplicate acknowledgment.

### 3. Duplicate Webhook

**Request** (intentionally matches the earlier sale payload):

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-1001",
  "created_at": "2026-06-08T10:15:00.000Z",
  "amount": 850,
  "currency": "NPR",
  "customer": { "phone": "9800000101" },
  "items": [{ "name": "Cappuccino", "qty": 1, "price": 850 }]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Event already processed"
}
```

**What Happened:** Samparka recognized the repeated delivery and acknowledged it safely without handling it as a new sale.

**What To Do Next:** Treat this response as success and stop retrying that payload.

### 4. Missing Binding (Blocked Location)

**Request** (against the token for a newly created integration before connect):

```json
{
  "event_type": "order.completed",
  "order_id": "system-sale-2001",
  "created_at": "2026-06-08T13:10:00.000Z",
  "amount": 600,
  "currency": "NPR",
  "customer": { "phone": "9800000103" },
  "items": [{ "name": "Latte", "qty": 1, "price": 600 }]
}
```

**Response:**

```json
{
  "success": false,
  "message": "Integration is not connected to a restaurant"
}
```

**What Happened:** Samparka resolved the integration from the webhook token, then rejected the event because no restaurant was connected yet.

**What To Do Next:** Connect the restaurant first, confirm the binding is stored on the integration, and then resend future events.

---

## Purchase QR Checkout (Communication)

> The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.

The Purchase QR endpoint lets **System terminals** create a Samparka *purchase-checkout* session on the fly and receive the QR link in the **same HTTP response**. The System renders that link as a QR and prints it on its receipt. The customer scans → WhatsApp flow → Wapio verification → points.

The endpoint reuses 100% of the existing purchase-checkout pipeline — no new models, no new loyalty logic, no new QR infrastructure.

### Endpoint

```
POST /integrations/system/{provider}/purchase-qr
```

**Auth:**

- `Authorization: Bearer <provider_api_key>` — confirms the System vendor (provider key `slug` must match the route `:provider`). The store is resolved from the API key's `store_id` scope.

No `webhook_token` is used on this endpoint. The provider API key confirms the vendor, and the key's `store_id` scope resolves the store and integration.

**Base URL:** `https://server.samparka.co` (all API requests).

**Prerequisites:**

- the System integration is `CONNECTED`/`ACTIVE`
- an active WhatsApp/Wapio communication provider is configured for the store

### Request

```bash
curl -X POST https://server.samparka.co/integrations/system/{provider}/purchase-qr \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <provider_api_key>" \
  -d '{
    "amount": 1250,
    "currency": "NPR",
    "items": [
      { "name": "Cappuccino", "qty": 1, "price": 850 },
      { "name": "Latte", "qty": 1, "price": 400 }
    ]
  }'
```

| Field | Type | Required | Description | Example |
| ----- | ---- | -------- | ----------- | ------- |
| `amount` | number | Yes | Transaction amount, must be greater than `0`. | `1250` |
| `currency` | string | No | 3-letter currency code. Defaults to `NPR` if omitted. | `NPR` |
| `items` | array | Yes | Product items in the bill. Each item has `name`, `qty`, and `price`. `items` total must equal `amount`. | `[{ "name": "Cappuccino", "qty": 1, "price": 850 }]` |

### Response — `200 Purchase QR ready`

```json
{
  "success": true,
  "message": "Purchase QR ready",
  "data": {
    "qr_link": "https://samparka.co/r/xKd93k",
    "purchase_reference": "ps_1690000000000_ab12cd34ef56",
    "amount": 1250,
    "currency": "NPR",
    "status": "QR_GENERATED",
    "expires_at": "2026-08-26T12:12:04.000Z",
    "integration_key": "xxxxxxxxxxxxxxxx"
  }
}
```

- `qr_link` is the short URL the System turns into a scannable QR and prints on the customer's receipt.
- A retried request while the same session is pending returns the same QR.

### Render QR On The Receipt

Turn `qr_link` into a QR image and print it on the customer's receipt. When the customer scans it, they are taken through the WhatsApp claim flow and points are awarded.

### End-To-End Sequence

```txt
 System                    Samparka                       Customer
   │                          │                              │
   │── POST { amount, items } ─▶│                            │
   │                          │                              │
   │                          │── Create purchase-checkout   │
   │                          │    session (Wapio signs QR)  │
   │                          │                              │
   │◀──────── { qr_link } ─────│                              │
   │                          │                              │
   │── Render QR and print    │                              │
   │    on the receipt        │                              │
   │                          │◀──────── scans QR ───────────│
   │                          │◀── WhatsApp deep link ───────│
   │                          │    & claims                  │
   │                          │                              │
   │                          │── Verification + completion  │
   │                          │                              │
   │                          │──────── points awarded ─────▶│
   │                          │                              │
```

### Error Codes

| Status | When |
|---|---|
| `400 purchase_qr_validation_failed` | Missing/non-positive `amount`, bad `currency`, missing/empty `items`, item missing `name` / non-positive `qty` / non-positive `price`, or `items` total ≠ `amount` |
| `401 invalid provider key` | Missing/invalid `Authorization: Bearer <provider_api_key>` |
| `403 store_scope_required` | API key not bound to a store |
| `404 unknown provider` | Provider not recognized or key's `provider_id` does not match the route `:provider` |
| `404 integration_not_found` | Store scope resolved but no `posintegrations` row for that provider |
| `409 system_not_connected` | The System integration status is not `CONNECTED`/`ACTIVE` — refused before any checkout is created |
| `422 not mapped to an outlet` | Integration has no `outlet_id` bound |
| `422 no active connected communication provider` | System is connected but no WhatsApp/Wapio communication provider is active — session creation aborts (no QR) |
| `409 already processed` | Session previously completed/claimed/failed |
| `410 expired` | Session previously expired |
| `502 QR generation failed` | Session created but `short_url` missing (Wapio/redirect failure) |

### Behavior Matrix

The endpoint only issues a QR when the System integration is connected **and** an active communication provider exists.

| System | WhatsApp/Wapio | Result |
|---|---|---|
| Not connected | anything | `409 system_not_connected` — no QR, no session |
| Connected | not connected | `422 no active connected communication provider` — no QR, no session |
| Connected | connected | `200` real Wapio QR |

### Idempotency

Each request creates a **new checkout session** (fresh `ps_…` `purchase_reference`) — there is no cross-request dedup. Retry-safety lives in the session lifecycle:

- `QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM` → retry returns the same QR (`200`).
- `COMPLETED` / `CLAIMED` / failed → retry returns `409` (`already processed`).
- Expired → retry returns `410` (`expired`).

Make a distinct request for each new sale. The `purchase_reference` uses the standard `ps_…` prefix (same as dashboard).

### Session Lifecycle

```txt
      ┌──────────┐
      ▼          │
[*] ──▶ CREATED  │
      │          │
      ▼          │
 QR_GENERATED ───┘  ◀── System retry while pending
      │                (same QR, HTTP 200)
      ▼
 WAITING_FOR_CUSTOMER_CLAIM  ◀── System retry while pending
      │                          (same QR, HTTP 200)
      ▼
 CLAIMED (customer scans)
      │
      ▼
 POINTS_AWARDED
      │
      ▼
 COMPLETED ──▶ [*]
```

Statuses that still return the same QR (retry-safe): `QR_GENERATED`, `WAITING_FOR_CUSTOMER_CLAIM`. Terminal/expired ones return `409`/`410`.

### Key Design Points

- **Auth via provider API key (Bearer) with store scope** — the Bearer key confirms the System vendor; the key's `store_id` scope resolves the store and integration. The API key must be bound to a store — a key without a store scope fails with `403 store_scope_required`. Reuses existing `authenticateProviderApiKey` — no new secrets, no webhook token in the path.
- **Fresh `purchase_reference` per request (no dedup)** — because the request carries no `bill_id`, each call creates a new session. Retry-safety lives in the session lifecycle, not across separate requests.
- **Items are required at creation** (`400` if missing) — the System must send product items so the QR session has the bill details needed for loyalty attribution.
- **`currency` defaults to `NPR`** — explicit value overrides.
- **System must be genuinely connected** — refuses with `409 system_not_connected` unless the integration status is `CONNECTED`/`ACTIVE`. A `CREATED`/`DISCONNECTED` integration can't issue checkouts.
- **Requires an active communication provider** — otherwise `422`; there is deliberately **no fallback** deep-link.
- **Location resolution** comes from the API key's `store_id` scope, not the webhook token:

```txt
Provider API Key (store_id scope)
-> SystemIntegration
-> Outlet
-> Bound External Location
```

### Purchase QR Testing

Prerequisites: System integration `CONNECTED`/`ACTIVE`, and an active WhatsApp/Wapio communication provider.

**Happy Path:**

```bash
curl -X POST "https://samparka.co/integrations/system/{provider}/purchase-qr" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <provider_api_key>" \
  --data '{
    "amount": 1250,
    "currency": "NPR",
    "items": [
      { "name": "Cappuccino", "qty": 1, "price": 850 }
    ]
  }'
```

Validate that:

- HTTP `200`
- `"success": true`
- `"status": "QR_GENERATED"`
- `qr_link` is a non-empty scannable short URL

**Idempotent Retry:** re-sending the request while the session is pending (`QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM`) returns the same `qr_link` (200), not a new session.

**Validation Failure:** send a body with a non-positive or missing `amount`, or missing/empty `items`. Expected:

```json
{
  "success": false,
  "message": "Invalid purchase QR request",
  "errors": { "code": "purchase_qr_validation_failed" }
}
```

**System Not Connected:** use an integration key whose System integration status is `CREATED`. Expected:

```json
{
  "success": false,
  "message": "{provider} is not connected for this store",
  "errors": { "code": "system_not_connected", "status": "CREATED" }
}
```

**No Active Communication Provider:** use a connected System integration without an active WhatsApp/Wapio communication provider. Expected:

```json
{
  "success": false,
  "message": "No active connected communication provider is configured"
}
```

**Invalid Token Or Unknown Provider:** expected `401` for an unknown/invalid token and `404` for an unknown `{provider}` value.

### Refunds For Purchase QR Sales

The Purchase QR flow does not have a separate refund mechanism. Once a checkout session is completed (customer scanned QR, verified, and points awarded), the loyalty outcome is managed through the same Samparka processing pipeline. If a reversal is needed for a Purchase QR sale, use the standard `refund.created` webhook event with the original sale identifier.

---

## Known Behaviors and Limitations

### Webhook Flow

- One Samparka integration key binds one System restaurant at a time.
- The supported connect contract is `integrationKey`, `externalLocationId`, and optional `externalLocationName` (legacy `restaurantId`/`restaurantName` aliases still accepted).
- Webhooks are delivered to `/webhook/{provider}/{token}` after the integration is connected.
- Restaurant attribution comes from the integration binding, not webhook payload restaurant fields.
- A `200` webhook acknowledgment confirms the delivery was accepted, not that loyalty activity was created.
- Duplicate webhook deliveries are accepted safely and can return `Event already processed`.
- Missing binding or disconnected integration state should be validated during testing.

### Purchase QR Flow

- Each request creates a **new checkout session** (fresh `ps_…` `purchase_reference`). There is no cross-request dedup — retry-safety lives in the session lifecycle.
- While a session is pending (`QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM`), retrying returns the same QR (`200`).
- A completed/claimed session rejects retries with `409`; an expired session rejects retries with `410`.
- The System must be `CONNECTED`/`ACTIVE` (not `CREATED`) to issue a QR.
- An active WhatsApp/Wapio communication provider is required — no fallback deep-link exists.
- `items` are required at creation time for loyalty attribution.
- `currency` defaults to `NPR` if omitted.

---

## Support Contact

Use the Samparka support channel already assigned to your integration rollout.

---

## OpenAPI And Postman

- Use `openapi.yaml` for machine-readable request and response definitions.
- Use `postman-collection.json` for hands-on testing — it already includes the partner API, webhook, and customer verification requests.