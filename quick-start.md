---
title: Quick Start
description: Root-level quick start for the outlet-owned RestroX integration.
sidebarTitle: Quick Start
---

Create the RestroX integration with:

```json
{
  "provider": "restrox",
  "storeId": "685000000000000000000010",
  "outletId": "685000000000000000000001"
}
```

Then verify this outcome flow:

```txt
Create Integration
↓
Connect Restaurant
↓
Submit Test Sale
↓
Verify Integration Became ACTIVE
↓
Verify Customer Exists
↓
Verify Loyalty Transaction Exists
↓
Verify Points Awarded
```
