---
title: Authentication
description: Root-level authentication summary for POS.
sidebarTitle: Authentication
---

- merchant lifecycle APIs use merchant auth
- partner APIs use `x-partner-key`, which Samparka shares manually during onboarding
- for this integration, use `restrox` as the `{provider}` value in documented partner and webhook routes
- POS customer APIs use `x-partner-key` or `Authorization: Bearer {{partnerKey}}`, plus `x-integration-key`
- webhooks use `POST /webhook/restrox/{token}`
