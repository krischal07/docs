---
title: Integration Checklist
description: Final go-live checklist for validating a RestroX to Samparka integration.
sidebarTitle: Integration Checklist
---


See also: [Testing Guide](./testing-guide).

## Connect Contract

- [ ] `POST /api/partners/restrox/connect` tested
- [ ] `integrationKey` verified
- [ ] `restaurantId` verified
- [ ] Bound restaurant stored on the integration
- [ ] Response status is `CONNECTED`
- [ ] Only the singular connect payload is used in active tooling and tests

