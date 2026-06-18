---
title: Known Behaviors and Limitations
description: Compatibility overview for the earlier POS known behaviors page, mapped to the current guide structure.
sidebarTitle: Known Behaviors
---

# Known Behaviors and Limitations

This page keeps the older POS URL working for existing bookmarks and search results.

The current POS guide no longer uses the older "native" information architecture. The supported behavior has been folded into the active onboarding, reliability, and troubleshooting pages below.

## What To Expect In The Current Flow

- One Samparka integration key binds one POS restaurant at a time.
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
If you arrived here from an older shared link, follow the pages above for the current partner-facing POS contract and go-live workflow.
</Info>
