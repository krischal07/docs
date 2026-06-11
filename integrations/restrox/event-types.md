---
title: Event Types
description: Review the RestroX webhook event types supported by Samparka.
sidebarTitle: Event Types
---

Samparka normalizes RestroX event payloads into these business events:

- `sale.completed`
- `refund.created`
- `sale.voided`

## Accepted Inputs

The mapper accepts these common RestroX strings:

| Input | Normalized Event |
| ------ | ------ |
| `order.completed` | `sale.completed` |
| `sale.completed` | `sale.completed` |
| `transaction.completed` | `sale.completed` |
| `refund.created` | `refund.created` |
| `refund.issued` | `refund.created` |
| `order.refund` | `refund.created` |
| `order.voided` | `sale.voided` |
| `order.void` | `sale.voided` |
| `order.cancelled` | `sale.voided` |
| `order.canceled` | `sale.voided` |
