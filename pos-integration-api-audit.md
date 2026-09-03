---
title: POS Integration Audit Notes
description: Root-level audit summary for the current POS contract.
sidebarTitle: Audit Notes
---

import { Tabs, Tab } from "@mintlify/components";

# POS Integration Audit

<Tabs>
  <Tab title="App">

> Evidence boundary: static audit of `samparka-backend/src` on 2026-08-11. This describes implemented code paths, not a claim that external providers or workers are running in production. Updated 2026-09-01 to reflect Purchase QR items-based flow and provider-agnostic endpoints.

## Architecture and entry points

POS is an adapter-based webhook ingress. `src/integrations/pos/providers/index.js` is the registry; it contains **Blanxer** and **RestroX**, each exposing `validate`, `parser.parse`, and `mapper.map`. `bootstrapProviders.js` seeds Provider records for Samparka, Blanxer, and RestroX. Provider records declare capabilities; the code rejects webhook use when the provider is not active/capable.

Public receipt endpoints are mounted twice: `POST /integrations/pos/:provider/events` and `/webhook/:provider/events`; the token routes are `POST /integrations/pos/restrox/:token`, `/blanxer/:token`, and `/:provider/:token`. The generic legacy provider route returns 410 unless `POS_LEGACY_PROVIDER_ROUTE_ENABLED=true`. The merchant-management API is under `/api/pos-integrations`; administration/replay surfaces are mounted under `/api/admin/pos` and provider administration under `/api/admin/pos/providers`. Partner lifecycle endpoints are `/api/partners/:provider/{connect,disconnect,test-sale}` (Bearer provider API key) and RestroX has `/api/partners/restrox/{connect,sync-locations,test-sale}`.

There is also a separate canonical ingress endpoint, `POST /api/v1/integrations/purchases`. It is not a third loyalty engine: Purchase Sessions use it to enter the same `InternalEvent -> processEvent` pipeline.

## Request-to-completion flow

1. `PosWebhookController._handle` generates a request ID, checks the named adapter, derives an idempotency key, and immediately writes a `RawWebhookEvent`. This happens before authentication, so failures are auditable (except an unknown provider, which is rejected before the write).
2. It checks provider capability, validates the request with the provider adapter, and resolves merchant routing. Token routes resolve `webhook_token` through `PosIntegrationLocation`; legacy routes resolve `(provider, x-api-key)` against `MerchantIntegration`/`PosIntegration`.
3. The resolved integration supplies `store_id`. Outlet-owned integrations use their bound `outlet_id`; store-owned integrations map the provider external location ID through `PosIntegrationLocation`. Missing, stale, disabled, non-participating, or unmapped locations produce an `InternalEvent` in `blocked_unmapped_location`, which is acknowledged but intentionally not processed.
4. Parser and mapper normalize the payload into provider-neutral `sale.completed`, `sale.voided`, or `refund.created`, with transaction, phone, source transaction ID, location snapshot, raw-event linkage, and trace IDs.
5. `ProcessedEvent` reserves the merchant-scoped idempotency slot before side effects. A new event becomes `InternalEvent`; a retryable failed reservation reuses its existing event. Then `eventProcessor.processEvent` runs processors in order.
6. Required `purchaseFactProcessor` transactionally creates or reuses a `PurchaseTransaction`; required `loyaltyProcessor` awards/reverses loyalty. Analytics and missions are explicitly registered as best-effort no-ops (`SKIPPED`) today. `InternalEvent` becomes `processed`, `failed_retryable`, or `failed_terminal`.

## Identity, validation, and protection

| Concern | Implemented behavior |
|---|---|
| Merchant | `MerchantIntegration.store_id`, found by token/API key; mapper output cannot select a store. |
| Outlet | Outlet-owned integration binding, otherwise `PosIntegrationLocation` external-location mapping and validity evaluation. |
| Customer | Provider-normalized E.164-ish `customer_phone`; loyalty resolves an existing store-scoped customer. Missing/invalid phone causes a loyalty skip, not customer creation. |
| Payload | adapter validator, strict parser, provider capability/state checks, integration/provider match, location checks. |
| Duplicate | `ProcessedEvent` unique index `(provider,pos_integration_id,idempotency_key)`. Key is event type + provider ID, or a SHA-256 fallback of selected transaction fields. |
| Race safety | reservation insert is the gate; `E11000` is treated as duplicate/in-flight/retry as appropriate. Ledger has a second unique `source.internal_event_id` gate. |
| Retry | retryable processor failures can be replayed by admin reprocess/replay services. The controller itself is synchronous; it does not enqueue POS processing. |

Webhook callers receive success acknowledgements for processed/duplicate/blocked events; malformed, unauthorized, disconnected, mismatched, and unready integrations receive 4xx. Processor failures are persisted and normally acknowledged only after the synchronous processor outcome is known. The processor never rethrows to a webhook caller; its catch marks terminal failure.

## State, collections, and operations

`RawWebhookEvent` is immutable payload audit storage with received/validated/normalized/processed/failed/duplicate status. `ProcessedEvent` is the durable reservation/dedup record (`reserved`, `processed`, `failed`). `InternalEvent` is the normalized domain input and processing/replay record. `EventProcessorState` stores a per-processor snapshot. `MerchantIntegration`/`PosIntegration` and `PosIntegrationLocation` represent connection and mapping; `Provider`/`ProviderApiKey` represent supported vendors and partner credentials. The downstream collections are `PurchaseTransaction`, `PurchaseItem`, `LoyaltyLedger`, `Customer`, and for L5 `CustomerOutletPoints` plus `L5PointsTransaction`.

The merchant controller/service manages connection records, location mappings, token/key rotation, sync, verification, activity, and webhook URLs. Admin controllers list raw/internal events, manually replay/reprocess, display mappings/metrics, and reconcile. `locationResolutionService` is the central mapping policy. `posWebhookService` delegates raw-event, reservation, and internal-event persistence to `IntegrationEventService`.

### Endpoint inventory

| Mount and endpoint | Auth / responsibility |
|---|---|
| `/integrations/pos/:provider/events` | Legacy adapter webhook; provider validator and `x-api-key`, feature-flagged. |
| `/integrations/pos/{restrox,blanxer}/:token`, `/:provider/:token` | Token-routed adapter webhooks; token resolves integration/location. |
| `/api/v1/integrations/purchases` | `integrationAuthMiddleware('submitPurchase')`; generic canonical purchase command. |
| `/api/pos-integrations/*` | `adminOrStoreMiddleware`; CRUD, locations, tokens/keys, status, verification, onboarding, sync/activity/share. |
| `/api/admin/pos/{raw-webhook-events,internal-events,pos-location-mappings,metrics,pos-reconciliation}` | admin middleware; inspection and replay/reprocess, not live POS ingress. |
| `/api/admin/pos/providers/*` | admin middleware; provider catalogue, provider keys, provider integrations/activity. |
| `/api/partners/:provider/{connect,disconnect,test-sale}` | Bearer provider API key; partner lifecycle. |
| `/api/partners/restrox/{connect,sync-locations,test-sale}` | RestroX-specific partner API. |
| `/api/partners/:provider/customers/{search,:customerId}` | provider customer-auth middleware; customer lookup by items. |
| `/integrations/pos/:provider/:token/purchase-qr` | Token-routed Purchase QR; creates scannable checkout session for cash sales. Returns QR link in same response. |

### Endpoint catalog — Communication tab

The endpoint catalog groups POS endpoints into **App** (webhook delivery, partner lifecycle, customer lookup) and **Communication** (Purchase QR) tabs. The Communication tab contains the Purchase QR endpoint and links to its overview, request/response contract, integration guide, and testing docs.

## Purchase QR — items-based flow

The Purchase QR endpoint was refactored to remove `bill_id` and `customer_phone` from the request. The POS now sends `amount`, `currency`, and `items` (product items with `name`, `qty`, `price`).

### Request shape

```json
{
  "amount": 1250,
  "currency": "NPR",
  "items": [
    { "name": "Cappuccino", "qty": 1, "price": 850 },
    { "name": "Latte", "qty": 1, "price": 400 }
  ]
}
```

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `amount` | number | Yes | Transaction amount, must be > 0. |
| `currency` | string | No | 3-letter currency code. Defaults to `NPR`. |
| `items` | array | Yes | Product items in the bill. Each item has `name`, `qty`, and `price`. |

### Key behavioral changes from previous `bill_id`/`phone` design

- `bill_id` is no longer sent or used as the idempotency key. The idempotency key is now derived from the request (provider + items).
- `customer_phone` is no longer required or sent. The QR is no longer a WhatsApp deep link targeting a specific phone number.
- Customer lookup for loyalty attribution happens downstream via the normal webhook processing pipeline, not at QR creation time.
- The response no longer includes `customer_phone` or `bill_id`.
- Error codes for invalid requests now check for missing/empty `items` instead of missing/invalid `customer_phone` or empty `bill_id`.

### Provider-agnostic language

All Purchase QR documentation now uses generic `:provider` path params and `{provider}` in examples instead of hardcoding `blanxer` or `restrox`. The endpoint works with any registered POS provider.

## Text sequence diagram

```text
POS -> PosWebhookController: POST token webhook
Controller -> RawWebhookEvent: insert audit record
Controller -> provider adapter: validate / parse / map
Controller -> locationResolutionService: resolve integration + outlet
Controller -> ProcessedEvent: atomic reservation
Controller -> InternalEvent: normalized event
Controller -> eventProcessor: processEvent
eventProcessor -> PurchaseTransaction: required transactional fact write
eventProcessor -> saleCompletedHandler: required loyalty write set
eventProcessor -> InternalEvent: status/outcome
Controller -> ProcessedEvent/RawWebhookEvent: processed
Controller --> POS: acknowledgement
```

## Important code units

- `PosWebhookController._handle`: inbound orchestration and acknowledgement.
- provider `validator/parser/mapper`: provider authentication, parsing, and canonical mapping.
- `locationResolutionService`: token/API-key and external-location resolution.
- `IntegrationEventService` and `PosWebhookService`: raw audit, reservation, and event persistence.
- `eventProcessor` / `internalEventProcessorRegistry`: processor lifecycle and required-vs-best-effort semantics.
- `purchaseFactProcessor` / `purchaseFactService`: transactional immutable purchase fact.
- `loyaltyProcessor`, `saleCompletedHandler`, `reversalEventHandler`: points and reversal handling.
- `replayService`, `reprocessBlockedEventService`: operator recovery; no automatic POS queue exists.

## Observed risks and boundaries

- The fallback idempotency hash uses a limited payload subset, so distinct sales with the same selected values can collide, while changed timestamps/amount representations can evade a duplicate.
- Raw-event storage occurs before authentication and can be abused for database growth absent the noted rate-limit placeholder.
- POS processing is request-synchronous, coupling provider latency to transactions and Mongo writes.
- Store-owned mapping can acknowledge blocked events; recovery is operational/manual.
- Analytics and mission processor registrations look complete but currently perform no work.
- No provider-webhook dead-letter queue is implemented; recovery is document/replay based.
- Purchase QR idempotency is now derived from request contents (provider + items) rather than an explicit `bill_id`, which means the collision properties depend on the item normalization in the idempotency hash.
  </Tab>
  <Tab title="Communication">
    <Info>
      The **Communication** feature requires a connected WhatsApp communication provider. To use this feature and get access, contact the Samparka team.
    </Info>

> Evidence boundary: static audit of `samparka-backend/src` on 2026-08-11. This describes implemented code paths, not a claim that external providers or workers are running in production. Updated 2026-09-01 to reflect Purchase QR items-based flow and provider-agnostic endpoints.

## Architecture and entry points

POS is an adapter-based webhook ingress. `src/integrations/pos/providers/index.js` is the registry; it contains **Blanxer** and **RestroX**, each exposing `validate`, `parser.parse`, and `mapper.map`. `bootstrapProviders.js` seeds Provider records for Samparka, Blanxer, and RestroX. Provider records declare capabilities; the code rejects webhook use when the provider is not active/capable.

Public receipt endpoints are mounted twice: `POST /integrations/pos/:provider/events` and `/webhook/:provider/events`; the token routes are `POST /integrations/pos/restrox/:token`, `/blanxer/:token`, and `/:provider/:token`. The generic legacy provider route returns 410 unless `POS_LEGACY_PROVIDER_ROUTE_ENABLED=true`. The merchant-management API is under `/api/pos-integrations`; administration/replay surfaces are mounted under `/api/admin/pos` and provider administration under `/api/admin/pos/providers`. Partner lifecycle endpoints are `/api/partners/:provider/{connect,disconnect,test-sale}` (Bearer provider API key) and RestroX has `/api/partners/restrox/{connect,sync-locations,test-sale}`.

There is also a separate canonical ingress endpoint, `POST /api/v1/integrations/purchases`. It is not a third loyalty engine: Purchase Sessions use it to enter the same `InternalEvent -> processEvent` pipeline.

## Purchase QR — items-based flow

The Purchase QR endpoint was refactored to remove `bill_id` and `customer_phone` from the request. The POS now sends `amount`, `currency`, and `items` (product items with `name`, `qty`, `price`).

### Request shape

```json
{
  "amount": 1250,
  "currency": "NPR",
  "items": [
    { "name": "Cappuccino", "qty": 1, "price": 850 },
    { "name": "Latte", "qty": 1, "price": 400 }
  ]
}
```

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `amount` | number | Yes | Transaction amount, must be > 0. |
| `currency` | string | No | 3-letter currency code. Defaults to `NPR`. |
| `items` | array | Yes | Product items in the bill. Each item has `name`, `qty`, and `price`. |

### Key behavioral changes from previous `bill_id`/`phone` design

- `bill_id` is no longer sent or used as the idempotency key. The idempotency key is now derived from the request (provider + items).
- `customer_phone` is no longer required or sent. The QR is no longer a WhatsApp deep link targeting a specific phone number.
- Customer lookup for loyalty attribution happens downstream via the normal webhook processing pipeline, not at QR creation time.
- The response no longer includes `customer_phone` or `bill_id`.
- Error codes for invalid requests now check for missing/empty `items` instead of missing/invalid `customer_phone` or empty `bill_id`.

### Provider-agnostic language

All Purchase QR documentation now uses generic `:provider` path params and `{provider}` in examples instead of hardcoding `blanxer` or `restrox`. The endpoint works with any registered POS provider.

## Observed risks and boundaries

- The fallback idempotency hash uses a limited payload subset, so distinct sales with the same selected values can collide, while changed timestamps/amount representations can evade a duplicate.
- Raw-event storage occurs before authentication and can be abused for database growth absent the noted rate-limit placeholder.
- POS processing is request-synchronous, coupling provider latency to transactions and Mongo writes.
- Store-owned mapping can acknowledge blocked events; recovery is operational/manual.
- Analytics and mission processor registrations look complete but currently perform no work.
- No provider-webhook dead-letter queue is implemented; recovery is document/replay based.
- Purchase QR idempotency is now derived from request contents (provider + items) rather than an explicit `bill_id`, which means the collision properties depend on the item normalization in the idempotency hash.
  </Tab>
</Tabs>
