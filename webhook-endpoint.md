---
title: Webhook Endpoint
description: Configure the RestroX webhook endpoint, request shape, and acknowledgment behavior.
sidebarTitle: Webhook Endpoint
---

# Webhook Endpoint

## Endpoint

- Method: `POST`
- Path: `/webhook/restrox/{token}`
- Content-Type: `application/json`

## What RestroX Needs To Send

Send a JSON object body with an `event_type` and the fields described in [Payload Reference](./payload-reference).

## What Response To Expect

Samparka returns JSON acknowledgments:

- `200 Event received`
- `200 Event already processed`
- `400 Request body must be a JSON object`
- `400 Missing event_type in payload`
- `404 Invalid webhook token`
- `500 Internal server error`

## Acknowledgment Timing

Samparka stores and validates the incoming request before returning the final acknowledgment for the canonical webhook path. A successful acknowledgment means the delivery was accepted. It does not guarantee that loyalty activity was created.

## What To Do If Something Goes Wrong

Use [Response Reference](./response-reference) and [Troubleshooting](./troubleshooting) to decide whether to retry, correct the payload, or confirm outlet mapping with Samparka.
