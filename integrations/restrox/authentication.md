---
title: Authentication
description: Root-level authentication summary for RestroX.
sidebarTitle: Authentication
---

- merchant lifecycle APIs use merchant auth
- partner APIs use `x-partner-key`
- RestroX customer APIs use `x-partner-key` or `Authorization: Bearer {{partnerKey}}`, plus `x-integration-key`
- webhooks use `POST /webhook/restrox/{token}`
