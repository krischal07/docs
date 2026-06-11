---
title: Idempotency
description: Understand duplicate webhook handling for RestroX.
sidebarTitle: Idempotency
---

Samparka accepts duplicate RestroX webhook deliveries safely.

```mermaid
sequenceDiagram
  participant R as RestroX
  participant S as Samparka
  R->>S: POST /webhook/restrox/{token}
  S-->>R: 200 Event received
  R->>S: POST /webhook/restrox/{token} (same event)
  S-->>R: 200 Event already processed
```

## Behavior

- duplicates return `200 Event already processed`
- idempotency is scoped to the integration
- sale and refund events with the same order identifier are treated as different event types
