---
title: Webhook Endpoint
description: Configure the RestroX webhook endpoint, request shape, and acknowledgment behavior.
sidebarTitle: Webhook Endpoint
---

## Endpoint

- Method: `POST`
- Path: `/webhook/restrox/{token}`
- Content-Type: `application/json`

## Webhook URL Aliases

RestroX webhook events may be delivered to either:

- `POST /webhook/restrox/{token}`
- `POST /integrations/pos/restrox/{token}`

Runtime verification confirmed both routes execute the same processing pipeline and produce identical outcomes. Partners should use the endpoint provided during onboarding.

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

## Payload Size

**Effective request size limit: 100 KB**

A global request body parser executes before route-level processing and enforces a 100 KB limit. Payloads larger than 100 KB are rejected before reaching webhook handling. Do not rely on a 1 MB limit.

## What To Do If Something Goes Wrong

Use [Response Reference](./response-reference) and [Troubleshooting](./troubleshooting) to decide whether to retry, correct the payload, or confirm outlet mapping with Samparka.
