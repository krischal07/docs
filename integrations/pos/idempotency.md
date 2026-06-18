---
title: Idempotency
description: Understand duplicate webhook handling and replay safety for POS events.
sidebarTitle: Idempotency
---


Samparka accepts duplicate webhook deliveries safely. If POS sends the same webhook more than once, Samparka can return `200 Event already processed` instead of creating duplicate downstream activity.

