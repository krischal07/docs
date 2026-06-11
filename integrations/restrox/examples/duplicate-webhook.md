---
title: Duplicate Webhook
description: Example showing safe duplicate handling for RestroX webhooks.
sidebarTitle: Duplicate Webhook
---

Send the same payload twice to the same webhook token.

Second response:

```json
{
  "success": true,
  "message": "Event already processed"
}
```
