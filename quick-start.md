---
title: Quick Start
description: Connect System to one Samparka outlet and validate partner and webhook delivery quickly.
sidebarTitle: Quick Start
---


<Tabs>
  <Tab title="App">
# Quick Start

This is the fastest path to a verified System integration.

See also: [Testing Guide](./testing-guide) and [Integration Checklist](./integration-checklist).

<Tip>
  Use the [Postman Collection](./reference/postman-collection) if you want the easiest setup path. It already includes the partner API, webhook, and customer verification requests.
</Tip>

## 1. Receive The Integration Key

Ask Samparka to manually share both of these values for your outlet-owned System integration:

- `integrationKey`
- provider API key to use as `Authorization: Bearer {{providerApiKey}}`

Use your provider slug in the `{provider}` route segment for all partner and webhook requests.

## 2. Connect The Location

Send `POST` requests to `/api/partners/{{provider}}/connect` with `Content-Type: application/json` and the partner auth header.

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

For backward compatibility, Samparka still accepts singular `restaurantId` and `restaurantName` fields and normalizes them to `externalLocationId` and `externalLocationName`. New integrations should send the generic location fields.

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

Do not assert `message`, `restaurantId`, or `externalLocationId` in the connect response. Those values are no longer part of the success payload.

## 3. Configure The Webhook URL

After a successful connect request, store the returned `token`.

Configure System to send webhook events to:

`https://samparka.xyz/webhook/{provider}/{{webhookToken}}`

## 4. Send A Test Sale

Use the canonical sale fixture from [`examples/payloads.json`](./examples/payloads.json):

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

Location identity comes from the integration that owns `{token}`. Do not rely on payload location fields for outlet-owned attribution.

## 5. Optional Partner Shortcut

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

## 6. Verify The Integration Became ACTIVE

Fetch the merchant integration after the first valid sale and confirm:

- `connectionStatus = ACTIVE`
- `healthStatus = HEALTHY`

`ACTIVE` confirms the integration activated, but it does not prove the loyalty workflow finished successfully.

## 7. Verify The Customer Exists

System authenticates as a partner, provides `Authorization: Bearer {{providerApiKey}}`, provides `x-integration-key`, searches the customer by phone, and receives customer loyalty data.

```bash
curl -X GET "https://your-domain/api/partners/{provider}/customers/search?phone={{customerPhone}}" \
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

## 8. Verify Customer Details And Points

Fetch the resolved customer:

```bash
curl -X GET "https://your-domain/api/partners/{provider}/customers/{{customerId}}" \
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

## 9. Verify Loyalty Transaction Exists

Use merchant tooling or the verified customer state to confirm a loyalty transaction was created for the test sale before go-live.

## 10. Repost The Same Payload Once

Send the exact same webhook body again. Samparka should acknowledge the duplicate safely.

Expected response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## 11. Send A Test Refund

Use a `refund.created` payload that reuses the original sale identifier.

Expected response:

```json
{
  "success": true,
  "message": "Event received"
}
```

## 12. Complete Go-Live Validation

Run the checks in [Integration Checklist](./integration-checklist) before switching to production traffic.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The fastest path to a working System-initiated QR checkout. The System requests a QR and prints it on the customer's receipt.

    ## Base URL

    All API requests are made to:

    ```text
    https://server.samparka.xyz
    ```

    ## Prerequisites

    Before calling the Purchase QR endpoint, confirm:

    - the System integration is `CONNECTED`/`ACTIVE`
    - an active WhatsApp/Wapio communication provider is configured for the store

    ## 1. Send A Purchase QR Request

    `POST /integrations/system/{provider}/purchase-qr`. Auth is the provider API key (`Authorization: Bearer <provider_api_key>`) plus the integration key (`X-Integration-Key` in request body). No `webhook_token` in the path.

    ```bash
    curl -X POST https://server.samparka.xyz/integrations/system/{provider}/purchase-qr \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer <provider_api_key>" \
      -d '{
        "amount": 1250,
        "currency": "NPR",
        "X-Integration-Key": "<integration_key>",
        "items": [
          { "name": "Cappuccino", "qty": 1, "price": 850 }
        ]
      }'
    ```

    ## 2. Receive The QR Link

    Expected response:

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
        "expires_at": "2026-08-26T12:12:04.000Z"
      }
    }
    ```

    ## 3. Render QR On The Receipt

    Turn `qr_link` into a QR image and print it on the customer's receipt. When the customer scans it, they are taken through the WhatsApp claim flow and points are awarded.

    ## 4. Idempotency

    Each request creates a **new checkout session** (fresh `ps_…` `purchase_reference`). Retry-safety lives in the session lifecycle: while the session is pending (`QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM`), retrying returns the same QR (200). See [Request / Response Contract](/integrations/system/purchase-qr/request-response) for the full behavior.

    <Columns cols={2}>
      <Card title="Integration Guide" icon="rocket" href="/integrations/system/purchase-qr/integration-guide">
        Full example curl, behavior matrix, and design choices.
      </Card>
      <Card title="Testing & Verification" icon="flask-conical" href="/integrations/system/purchase-qr/testing">
        Automated test cases and deployment notes.
      </Card>
    </Columns>
  </Tab>
</Tabs>
