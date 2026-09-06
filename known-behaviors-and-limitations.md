---
title: Known Behaviors and Limitations
description: Compatibility overview for the earlier System known behaviors page, mapped to the current guide structure.
sidebarTitle: Known Behaviors
---


<Tabs>
  <Tab title="App">
# Known Behaviors and Limitations

This page keeps the older System URL working for existing bookmarks and search results.

The current System guide no longer uses the older "native" information architecture. The supported behavior has been folded into the active onboarding, reliability, and troubleshooting pages below.

## What To Expect In The Current Flow

- One Samparka integration key binds one System restaurant at a time.
- The supported connect contract is `integrationKey`, `restaurantId`, and optional `restaurantName`.
- Webhooks are delivered to `/webhook/{provider}/{token}` after the integration is connected.
- Restaurant attribution comes from the integration binding, not webhook payload restaurant fields.
- A `200` webhook acknowledgment confirms the delivery was accepted, not that loyalty activity was created.
- Duplicate webhook deliveries are accepted safely and can return `Event already processed`.
- Missing binding or disconnected integration state should be validated during testing.

## Use These Current Pages

- [Quick Start](./quick-start) for the active connect and webhook sequence
- [Restaurant Binding Reference](./location-mapping) for restaurant attribution expectations
- [Idempotency](./idempotency) for replay-safe webhook handling
- [Troubleshooting](./troubleshooting) for validation failures and delivery issues

<Info>
If you arrived here from an older shared link, follow the pages above for the current partner-facing System contract and go-live workflow.
</Info>
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR endpoint has these known behaviors:

    - Each request creates a **new checkout session** (fresh `ps_…` `purchase_reference`). There is no cross-request dedup — retry-safety lives in the session lifecycle.
    - While a session is pending (`QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM`), retrying returns the same QR (`200`).
    - A completed/claimed session rejects retries with `409`.
    - An expired session rejects retries with `410`.
    - The System must be `CONNECTED`/`ACTIVE` (not `CREATED`) to issue a QR.
    - An active WhatsApp/Wapio communication provider is required — no fallback deep-link exists.
    - `items` are required at creation time for loyalty attribution.
    - `currency` defaults to `NPR` if omitted.

    <Columns cols={2}>
      <Card title="Overview" icon="compass" href="/integrations/system/purchase-qr/overview">
        Architecture, state lifecycle, and end-to-end sequence.
      </Card>
      <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
        Full error codes and idempotency behavior.
      </Card>
    </Columns>
  </Tab>
</Tabs>
