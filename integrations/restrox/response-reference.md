---
title: Response Reference
description: Partner-visible responses for RestroX webhook and partner flows.
sidebarTitle: Response Reference
---

## Webhook Responses

| Status | Message | Meaning |
| ------ | ------- | ------- |
| `200` | `Event received` | Valid event accepted. |
| `200` | `Event already processed` | Duplicate request accepted idempotently. |
| `400` | `Request body must be a JSON object` | Invalid body shape. |
| `400` | `Missing event_type in payload` | Required event type absent. |
| `404` | `Invalid webhook token` | Token did not resolve to a RestroX integration. |
| `409` | `Integration disconnected` | Integration lifecycle status is `DISCONNECTED`. |
| `409` | `Integration is not connected to a restaurant` | Integration has no stored restaurant binding. |
| `409` | `Restaurant binding mismatch` | Payload restaurant identifier does not match the stored binding. |

## Partner API Responses

| Route | Status | Meaning |
| ------ | ------ | ------- |
| `connect` | `200` | RestroX connected successfully. |
| `connect` | `409` | Duplicate binding or rebind attempted before disconnect. |
| `sync-locations` | `200` | Route exists but returns skipped for outlet-owned RestroX. |
| `test-sale` | `200` | Test sale accepted into webhook pipeline. |
| `test-sale` | `409` | Stored restaurant binding is missing or does not match. |
