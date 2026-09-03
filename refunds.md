---
title: Refunds
description: Send refund events correctly so Samparka can reverse prior loyalty activity.
sidebarTitle: Refunds
---


<Tabs>
  <Tab title="App">
# Refunds

Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should reference the original sale identifier that was already sent to Samparka.

See also: [Event Types](./event-types) and [Testing Guide](./testing-guide).
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

    The Purchase QR flow does not have a separate refund mechanism. Once a checkout session is completed (customer scanned QR, verified, and points awarded), the loyalty outcome is managed through the same Samparka processing pipeline.

    If a reversal is needed for a Purchase QR sale, use the standard `refund.created` webhook event with the original sale identifier.

    <Columns cols={2}>
      <Card title="Request / Response Contract" icon="code" href="/integrations/pos/purchase-qr/request-response">
        Purchase QR session lifecycle and completion states.
      </Card>
      <Card title="Integration Guide" icon="rocket" href="/integrations/pos/purchase-qr/integration-guide">
        Example curl calls and behavior matrix.
      </Card>
    </Columns>
  </Tab>
</Tabs>
