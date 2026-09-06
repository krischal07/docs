---
title: Refunds
description: Send refund events correctly so Samparka can reverse prior loyalty activity.
sidebarTitle: Refunds
---

import { Tabs, Tab } from "@mintlify/components";

<Tabs>
  <Tab title="App">
Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should reference the original sale identifier that was already sent to Samparka.

See also: [Event Types](./event-types) and [Testing Guide](./testing-guide).
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

Samparka accepts `refund.created` and `order.voided` partner events for reversal scenarios. For successful reversal handling, the refund or void should reference the original sale identifier that was already sent to Samparka.

See also: [Event Types](./event-types) and [Testing Guide](./testing-guide).

<Columns cols={2}>
  <Card title="Request / Response Contract" icon="code" href="/integrations/system/purchase-qr/request-response">
    Full error codes and idempotency behavior.
  </Card>
  <Card title="Integration Guide" icon="rocket" href="/integrations/system/purchase-qr/integration-guide">
    Example curl calls and behavior matrix.
  </Card>
</Columns>
  </Tab>
</Tabs>
