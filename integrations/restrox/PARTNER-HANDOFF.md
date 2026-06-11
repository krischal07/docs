---
title: RestroX Partner Handoff
description: Canonical handoff summary for the outlet-owned RestroX launch architecture.
sidebarTitle: Partner Handoff Summary
---

# RestroX Partner Handoff

RestroX is an outlet-owned integration. The onboarding sequence is:

```mermaid
flowchart TD
  A["Merchant creates integration"] --> B["Samparka generates Integration Key"]
  B --> C["Samparka generates webhook token"]
  C --> D["RestroX connects restaurant"]
  D --> E["Samparka stores external_location_id"]
  E --> F["RestroX sends first sale"]
  F --> G["Integration becomes ACTIVE"]
```

## What To Share

- outlet-specific Integration Key
- outlet-specific webhook URL
- expected restaurant identifier for that outlet

## What To Avoid

- do not plan a migration flow
- do not create a second RestroX integration for the same outlet
- do not use store-owned setup assumptions for RestroX
