---
title: Integration Checklist
description: Final go-live checklist for validating a POS to Samparka integration.
sidebarTitle: Integration Checklist
---


# Integration Checklist

See also: [Testing Guide](./testing-guide).

## Connect Contract

- [ ] `POST /api/partners/{provider}/connect` tested
- [ ] `integrationKey` verified
- [ ] `externalLocationId` verified
- [ ] Bound external location stored on the integration
- [ ] Response status is `CONNECTED`
- [ ] Only the location-based connect payload is used in active tooling and tests
