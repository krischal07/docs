---
title: Testing Guide
description: Validate the outlet-owned RestroX lifecycle and webhook contract.
sidebarTitle: Testing Guide
---

Use this sequence when testing RestroX end to end.

## 1. Create The Integration

Confirm the merchant API returns:

- `ownershipMode = OUTLET_OWNED`
- `status = CREATED`
- `connectionStatus = CREATED`
- `integrationKey`
- `webhookUrl`

## 2. Connect The Restaurant

Confirm the partner connect route returns:

- `connected = true`
- `status = CONNECTED`
- `externalLocationId`

## 3. Send The First Sale

Send a valid sale to the webhook URL and then fetch integration status.

Expected result:

- webhook transport response `200 Event received`
- integration lifecycle status becomes `ACTIVE`
- health status becomes `HEALTHY`

## 4. Send A Mismatch Sale

Use the same token but a different restaurant identifier.

Expected result:

- webhook response `409 Restaurant binding mismatch`
- integration health status becomes `ERROR`

## 5. Disconnect, Reconnect, Rebind

Expected results:

- delete on outlet-owned integration returns `disconnected = true`
- reconnect returns `status = CONNECTED`
- reconnect keeps the same webhook token and Integration Key
- rebind is only allowed after disconnect
