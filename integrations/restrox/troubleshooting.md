---
title: Troubleshooting
description: Diagnose common outlet-owned RestroX integration issues.
sidebarTitle: Troubleshooting
---

| Symptom | Likely Cause | What To Check |
| ------ | ------- | ------- |
| Create integration fails | Store has no outlets or outlet is invalid | Create an outlet first and verify `outletId` belongs to the store. |
| Connect returns `Invalid Integration Key` | Wrong Integration Key | Copy the Integration Key from the merchant integration detail again. |
| Connect returns duplicate binding conflict | Restaurant already bound elsewhere | Check whether another RestroX outlet already owns that restaurant identifier. |
| Webhook returns `Invalid webhook token` | Wrong token | Verify the integration-level webhook URL. |
| Webhook returns `Restaurant binding mismatch` | Payload restaurant identifier changed | Confirm `restaurantId` matches the bound `external_location_id`. |
| No activation after a sale | Sale did not reach the correct binding | Fetch integration status and confirm `connectionStatus`, `healthStatus`, and `externalLocationId`. |
