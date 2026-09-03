---
title: Integration Checklist
description: Final go-live checklist for validating a POS to Samparka integration.
sidebarTitle: Integration Checklist
---


<Tabs>
  <Tab title="App">
# Integration Checklist

See also: [Testing Guide](./testing-guide).

## Connect Contract

- [ ] `POST /api/partners/{provider}/connect` tested
- [ ] `integrationKey` verified
- [ ] `externalLocationId` verified
- [ ] Bound external location stored on the integration
- [ ] Response status is `CONNECTED`
- [ ] Only the location-based connect payload is used in active tooling and tests
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    ## Purchase QR Checkout

    See [Purchase QR Integration Guide](/integrations/pos/purchase-qr/integration-guide).

    - [ ] `POST /integrations/pos/{provider}/purchase-qr` tested with a valid sale
    - [ ] Response `200` with `status = QR_GENERATED` and a non-empty `qr_link`
    - [ ] `qr_link` renders as a scannable QR and applies to the customer
    - [ ] Same request retried returns the same QR (idempotent)
    - [ ] POS integration status is `CONNECTED`/`ACTIVE` (not `CREATED`)
    - [ ] Active WhatsApp/Wapio communication provider is configured for the store
    - [ ] Validation failure (`400`), POS-not-connected (`409`), and no-comm-provider (`422`) paths observed

    <Columns cols={2}>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Full error codes and idempotency behavior.
      </Card>
      <Card title="Testing & Verification" icon="flask-conical" href="/integrations/pos/purchase-qr/testing">
        Automated test cases and deployment notes.
      </Card>
    </Columns>
  </Tab>
</Tabs>
