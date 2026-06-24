---
title: Authentication
description: Root-level authentication summary for POS.
sidebarTitle: Authentication
---

- merchant lifecycle APIs use merchant auth
- partner APIs use `Authorization: Bearer {{providerApiKey}}` as the canonical auth model
- for this integration, use `restrox` as the `{provider}` value in documented partner and webhook routes
- POS customer APIs use `Authorization: Bearer {{providerApiKey}}` plus `x-integration-key`
- `x-partner-key` remains legacy compatibility behavior and should not be used in new integrations
- webhooks use `POST /webhook/restrox/{token}`
