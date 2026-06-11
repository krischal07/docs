---
title: Implementation Notes
description: Notes captured while aligning RestroX docs to the outlet-owned backend behavior.
sidebarTitle: Implementation Notes
---

This page now records source-of-truth findings instead of historical gap analysis.

## Verified Implementation Facts

- `PosIntegration` stores RestroX restaurant bindings directly
- `ownership_mode` is `OUTLET_OWNED` for RestroX
- webhook tokens are stored on the integration for outlet-owned RestroX
- connect sets lifecycle status to `CONNECTED`
- first valid sale promotes lifecycle status to `ACTIVE`
- disconnect sets lifecycle status to `DISCONNECTED`
