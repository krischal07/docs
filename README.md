---
title: RestroX Partner Guide
description: Canonical overview of the RestroX webhook integration with Samparka Loyalty.
sidebarTitle: Guide Overview
---

# RestroX Partner Guide

Samparka is a customer loyalty platform. The RestroX webhook integration lets RestroX send completed-sale, refund, and void events to Samparka so eligible transactions can be evaluated for loyalty activity. RestroX sends JSON webhooks to a token-based URL, and Samparka responds with a simple success or error message for each delivery.

## Integration Flow

1. Receive the Samparka webhook URL and token.
2. Configure RestroX to send `POST` requests with a JSON body.
3. Send a test `order.completed` webhook.
4. Confirm a `200` acknowledgment.
5. Repost the same payload once to confirm duplicate safety.
6. Send a `refund.created` webhook that references the original sale identifier.
7. Complete the go-live checklist.

## Quick Links

- [Partner Handoff](./PARTNER-HANDOFF)
- [Quick Start](./quick-start)
- [Authentication](./authentication)
- [Webhook Endpoint](./webhook-endpoint)
- [Event Types](./event-types)
- [Payload Reference](./payload-reference)
- [Response Reference](./response-reference)
- [Refunds](./refunds)
- [Idempotency](./idempotency)
- [Location Mapping](./location-mapping)
- [Testing Guide](./testing-guide)
- [Troubleshooting](./troubleshooting)
- [Endpoint Catalog](./endpoint-catalog)
- [Integration Checklist](./integration-checklist)
- [OpenAPI](./openapi.yaml)
- [Postman Collection](./postman-collection.json)

## Important Integration Note

A `200 Event received` response means Samparka accepted the webhook delivery. It does not guarantee that loyalty activity was created, because customer and location requirements are checked after the request is accepted.
