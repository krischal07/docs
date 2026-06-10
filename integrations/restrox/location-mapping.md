---
title: Location Mapping
description: Review the webhook-facing location requirements used after native onboarding and location sync are complete.
sidebarTitle: Location Mapping
---

This page documents the webhook-facing location rules used by the RestroX event pipeline after native onboarding and location sync have established the integration context.

<Note>
For the native location sync and review workflow, use [Store Linking](./native/store-linking).
</Note>

## What RestroX Needs To Send

Send one of these location fields in event payloads:

- `external_location_id`
- `location_id`
- `outlet_id`
- `branch_id`

`external_location_name`, `location_name`, `branch_name`, or `outlet_name` can also be sent as optional labels.

## How This Fits The Native Flow

The native partner sync flow attempts to link RestroX locations to Samparka outlets before production traffic begins.

Webhook event delivery still depends on those linked location identifiers. A valid transport response does not guarantee the payload location was processable.

## What Response To Expect

If the webhook token is valid, Samparka can still return `200 Event received` even when the payload location does not match a processable location configuration.

## What To Do If Something Goes Wrong

If a sale is accepted but expected loyalty activity is missing, verify:

1. The payload includes a location identifier.
2. The location identifier matches the synced RestroX location value.
3. The matched location is active and not waiting for review.

## Related Documentation

- [Store Linking](./native/store-linking)
- [Webhook Endpoint](./webhook-endpoint)
- [Troubleshooting](./troubleshooting)
