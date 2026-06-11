---
title: Endpoint Catalog
description: Endpoint inventory for the current RestroX contract.
sidebarTitle: Endpoint Catalog
---

## Merchant Integration APIs

| Method | Path | Purpose |
| ------ | ---- | ------- |
| `POST` | `/api/pos-integrations` | Create outlet-owned RestroX integration |
| `GET` | `/api/pos-integrations/{id}` | Fetch integration detail |
| `PATCH` | `/api/pos-integrations/{id}` | Update integration metadata or outlet before connect / after disconnect |
| `DELETE` | `/api/pos-integrations/{id}` | Disconnect outlet-owned RestroX integration |
| `GET` | `/api/pos-integrations/{id}/status` | Fetch derived status and readiness summary |
| `POST` | `/api/pos-integrations/{id}/verify` | Fetch production-readiness summary |
| `GET` | `/api/pos-integrations/{id}/webhook-url` | Fetch integration webhook URL |
| `POST` | `/api/pos-integrations/{id}/integration-key/rotate` | Rotate Integration Key |

## Merchant APIs Not Used By Outlet-Owned RestroX

These routes still exist in the shared POS surface, but outlet-owned RestroX treats them as not applicable and returns `409` when they require store-owned behavior:

| Method | Path |
| ------ | ---- |
| `GET` | `/api/pos-integrations/{id}/locations` |
| `PUT` | `/api/pos-integrations/{id}/locations` |
| `POST` | `/api/pos-integrations/{id}/sync-outlets` |

## Partner APIs

| Method | Path | Purpose |
| ------ | ---- | ------- |
| `POST` | `/api/partners/restrox/connect` | Bind a restaurant to the outlet-owned integration |
| `POST` | `/api/partners/restrox/sync-locations` | Returns skipped for outlet-owned RestroX |
| `POST` | `/api/partners/restrox/test-sale` | Submit a test sale into the webhook processing path |
| `GET` | `/api/partners/{provider}/customers/search` | Search customer by exact phone |
| `GET` | `/api/partners/{provider}/customers/{customerId}` | Fetch one customer |

## Webhook

| Method | Path | Purpose |
| ------ | ---- | ------- |
| `POST` | `/webhook/restrox/{token}` | Receive sale, refund, and void events |
| `POST` | `/integrations/pos/restrox/{token}` | Verified alias that runs the same token-routed pipeline |
