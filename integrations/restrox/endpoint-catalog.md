---
title: Endpoint Catalog
description: Endpoint inventory for the RestroX partner connect and webhook integration.
sidebarTitle: Endpoint Catalog
---

# Endpoint Catalog

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/api/partners/restrox/connect` | Bind one RestroX restaurant to one outlet-owned Samparka integration | `x-partner-key` header | Active |
| `POST` | `/webhook/restrox/{token}` | Receive RestroX sale, refund, and void webhooks | Token in URL path | Active |

Source:
samparka-backend/src/index.js:146-155
samparka-backend/src/integrations/pos/partners/restrox/routes.js:10-12
samparka-backend/src/integrations/pos/routes.js:11-15
