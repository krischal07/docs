---
title: Idempotency
description: Understand duplicate webhook handling and replay safety for RestroX events.
sidebarTitle: Idempotency
---

# Idempotency

Samparka accepts duplicate webhook deliveries safely. If RestroX sends the same webhook more than once, Samparka can return `200 Event already processed` instead of creating duplicate downstream activity.

