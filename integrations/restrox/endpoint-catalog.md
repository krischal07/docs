---
title: Endpoint Catalog
description: Endpoint inventory for the RestroX partner connect and webhook integration.
sidebarTitle: Endpoint Catalog
---

# Endpoint Catalog

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/api/pos-integrations` | Create an outlet-owned RestroX integration | Merchant auth | Active |
| `GET` | `/api/pos-integrations/{id}` | Fetch the merchant integration and verify lifecycle state | Merchant auth | Active |
| `DELETE` | `/api/pos-integrations/{id}` | Disconnect an outlet-owned RestroX integration | Merchant auth | Active |
| `POST` | `/api/partners/restrox/connect` | Bind one RestroX restaurant to one outlet-owned Samparka integration | `x-partner-key` header | Active |
| `POST` | `/api/partners/restrox/test-sale` | Submit a partner-side test sale into the webhook pipeline | `x-partner-key` header | Active |
| `POST` | `/webhook/restrox/{token}` | Receive RestroX sale, refund, and void webhooks | Token in URL path | Active |
| `GET` | `/api/customers/search` | Search for the customer created or resolved by the test sale | Merchant auth | Active |
| `GET` | `/api/customers/{customerId}` | Verify loyalty data and points after sale processing | Merchant auth | Active |
| `POST` | `/api/partners/restrox/sync-locations` | Compatibility-only location sync check | `x-partner-key` header | Deprecated |
| `GET` | `/api/pos-integrations/{id}/locations` | Compatibility-only shared locations route | Merchant auth | Deprecated |

Source:
samparka-backend/src/index.js:146-155
samparka-backend/src/integrations/pos/partners/restrox/routes.js:10-12
samparka-backend/src/integrations/pos/routes.js:11-15
