---
title: Endpoint Catalog
description: Endpoint inventory for the RestroX webhook integration.
sidebarTitle: Endpoint Catalog
---

# Endpoint Catalog

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/webhook/restrox/{token}` | Receive RestroX sale, refund, and void webhooks | Token in URL path | Active |

Source:
samparka-backend/src/index.js:151-155
samparka-backend/src/integrations/pos/routes.js:11-15
