---
title: Testing Guide
description: Root-level testing summary for the outlet-owned RestroX flow.
sidebarTitle: Testing Guide
---

Test in this order:

1. create integration
2. connect restaurant
3. send first valid sale
4. verify `ACTIVE`
5. send mismatch webhook
6. verify `ERROR`
