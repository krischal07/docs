---
title: Idempotency
description: Understand duplicate webhook handling and replay safety for POS events.
sidebarTitle: Idempotency
---


Samparka accepts duplicate webhook deliveries safely. If POS sends the same webhook more than once, Samparka can return `200 Event already processed` instead of creating duplicate downstream activity.

## Purchase QR Idempotency

The purchase checkout endpoint is also idempotent. See [Request / Response Contract](./purchase-qr/request-response).

The idempotency key is `posqr:{provider}:{sanitized(bill_id)}`. The same `provider + bill_id` for the same merchant always maps to the same session, so a retried bill reuses the pending session's QR instead of creating a duplicate.

- `QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM` → retry returns the same QR (`200`).
- `COMPLETED` / `CLAIMED` / failed → retry returns `409` (`already processed`).
- Expired → retry returns `410` (`expired`).

Use a new `bill_id` for each new sale.

