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

## Purchase QR Checkout

See [Purchase QR Integration Guide](./purchase-qr/integration-guide).

- [ ] `POST /integrations/pos/{provider}/purchase-qr` tested with a valid sale
- [ ] Response `200` with `status = QR_GENERATED` and a non-empty `qr_link`
- [ ] `qr_link` renders as a scannable QR and applies to the customer
- [ ] Same request retried returns the same QR (idempotent)
- [ ] POS integration status is `CONNECTED`/`ACTIVE` (not `CREATED`)
- [ ] Active WhatsApp/Wapio communication provider is configured for the store
- [ ] Validation failure (`400`), POS-not-connected (`409`), and no-comm-provider (`422`) paths observed
