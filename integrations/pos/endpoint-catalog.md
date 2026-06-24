---
title: Endpoint Catalog
description: Root-level endpoint summary for POS.
sidebarTitle: Endpoint Catalog
---

| Endpoint | Purpose | Authentication |
| -------- | ------- | -------------- |
| `POST /api/partners/{provider}/connect` | Bind one external location to one outlet-owned Samparka integration. | `Authorization: Bearer {{providerApiKey}}` |
| `POST /api/partners/{provider}/test-sale` | Submit a partner-side test sale into the webhook processing path. | `Authorization: Bearer {{providerApiKey}}` |
| `POST /webhook/{provider}/{token}` | Deliver sale, refund, or void events to the tokenized webhook endpoint. | Tokenized URL path |
| `GET /api/partners/{provider}/customers/search` | Search for one customer by phone within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
| `GET /api/partners/{provider}/customers/{customerId}` | Fetch one customer record within one integration scope. | `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key` |
