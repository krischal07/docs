---
title: Response Reference
description: Reference the partner-visible response contracts for connect, test-sale, customer APIs, and webhooks.
sidebarTitle: Response Reference
---


These are the partner-visible responses for the active RestroX integration path.

See also: [Troubleshooting](./troubleshooting).

## Connect Success

```json
{
  "success": true,
  "message": "RestroX connected",
  "connected": true,
  "integrationId": "684915c401ec340433d84ee2",
  "token": "abcxyz",
  "restaurantId": "ktm-branch-01",
  "externalLocationId": "ktm-branch-01",
  "externalLocationName": "Kathmandu Branch",
  "status": "CONNECTED"
}
```

## Test Sale Success

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

## Test Sale Duplicate

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

## Customer Search Hit

```json
{
  "exists": true,
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "restrox-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

## Customer Search Miss

```json
{
  "exists": false
}
```

## Customer Detail

```json
{
  "customer": {
    "id": "684a00000000000000001001",
    "name": null,
    "phone": "9800000101",
    "email": "restrox-sale-1001@example.com",
    "points": 85,
    "tier": null,
    "lifetimePoints": 85,
    "membershipSince": "2026-06-08T10:15:00.000Z"
  }
}
```

## `200 Event received`

```json
{
  "success": true,
  "message": "Event received"
}
```

## `200 Event already processed`

```json
{
  "success": true,
  "message": "Event already processed"
}
```

## `400 Request body must be a JSON object`

```json
{
  "success": false,
  "message": "Request body must be a JSON object"
}
```

## `400 Missing event_type in payload`

```json
{
  "success": false,
  "message": "Missing event_type in payload"
}
```

## `404 Invalid webhook token`

```json
{
  "success": false,
  "message": "Invalid webhook token"
}
```

## `409 Integration disconnected`

```json
{
  "success": false,
  "message": "Integration disconnected"
}
```

## `409 Integration is not connected to a restaurant`

```json
{
  "success": false,
  "message": "Integration is not connected to a restaurant"
}
```

## `409 Integration is not ready to receive webhooks`

```json
{
  "success": false,
  "message": "Integration is not ready to receive webhooks"
}
```

## Partner Customer Error Envelope

```json
{
  "error": "partner_customer_error",
  "message": "Phone is required"
}
```

The same `{ error, message }` envelope is used for partner customer validation, auth, and lookup failures.
