---
title: Documentation Gap Analysis Report
description: Audit findings comparing RestroX integration documentation against the actual implementation.
sidebarTitle: Gap Analysis Report
---

This report documents the findings from a full audit of the RestroX integration documentation against the production backend implementation.

---

## Files Modified

| File | Change Type |
| ---- | ----------- |
| `native/authentication.mdx` | Bug fix — wrong error codes |
| `native/error-codes.mdx` | Bug fix — wrong error code; reordered 503 entry |
| `native/store-linking.mdx` | Bug fix — removed non-existent statuses; improved status descriptions |
| `native/merchant-onboarding.mdx` | Correction — clarified `CONNECTED` status is unreachable |
| `native/loyalty-processing.mdx` | New content — unknown event type fallback documented |
| `native/customer-identity.mdx` | New content — NP default country assumption for phone normalization |
| `native/partner-api.mdx` | New content — 503 error condition added |
| `payload-reference.md` | Correction — restructured Required vs Recommended vs Optional; clarified `200` behavior when fields absent |
| `event-types.md` | New content — full alias table and unknown event type fallback |
| `idempotency.md` | New content — key construction format, event_type prefix, per-integration scope |
| `refunds.md` | Clarification — exact matching requirement explained |
| `response-reference.md` | New content — all blocked reason codes documented |
| `troubleshooting.md` | New content — missing blocking scenarios added |
| `integration-checklist.md` | New content — native onboarding, customer lookup, and idempotency checks added |

## Files Created

| File | Purpose |
| ---- | ------- |
| `gap-analysis-report.md` | This report |

---

## Critical Bugs Fixed

### 1. Wrong Error Codes in `authentication.mdx`

**Problem:** Three error code values were wrong for partner customer authentication failures.

| Failure | Documented (wrong) | Actual |
| ------- | ------------------ | ------ |
| Missing integration key | `partner_customer_error` | `unauthorized` |
| Invalid integration key | `partner_customer_error` | `unauthorized` |
| Integration inactive | `partner_customer_error` | `forbidden` |

**Source:** `customerAuthMiddleware.js` lines 36–49.

---

### 2. Wrong Error Code in `error-codes.mdx`

**Problem:** Integration inactive row had code `partner_customer_error`. Actual middleware returns `"forbidden"`.

**Source:** `customerAuthMiddleware.js` line 49.

---

### 3. `payload-reference.md` — Required Fields Incorrectly Marked

**Problem:** Both `external_location_id` and `order_id`/`transaction_id`/`id` were marked `Yes` in the Required column. Neither causes an HTTP error when absent.

- **`event_type`** is the only field that causes a `400` response when absent.
- **`order_id`** absence causes a SHA-256 fallback idempotency key — no `400`.
- **`external_location_id`** absence causes the event to be stored with `blocked_unmapped_location` status and `200` returned — no `400`.

**Fix:** Restructured into Required / Strongly Recommended / Optional sections with accurate behavior notes.

---

### 4. `store-linking.mdx` — Non-Existent Location Statuses

**Problem:** `mapped` and `disabled` were listed as verified location row statuses. Neither value appears in the production implementation.

**Actual statuses in use:** `draft`, `active`.
**Actual lifecycle statuses:** `ACTIVE`, `STALE`.

**Source:** `restrox/service.js` (`syncLocations` function) and `merchant/service.js` (`syncOutlets` function).

---

### 5. `testing-guide.md` — Wrong Location Test Had Wrong Expected Response

**Problem:** The "Wrong Location Test" section showed a `404` error response. A direct webhook delivery with an unknown location returns `200 Event received` (the event is accepted but blocked internally). The `404` response only comes from the `test-sale` endpoint.

**Fix:** Split into two subsections — one for `test-sale` (correct `404`), one for direct webhook (correct `200`).

---

## Missing Behaviors Now Documented

### 6. Event Type Aliases

**Gap:** Only `order.completed`, `refund.created`, and `order.voided` were documented. The mapper accepts many additional aliases that are used in practice.

**Added to `event-types.md`:**

| Alias | Maps to |
| ----- | ------- |
| `transaction.completed` | `sale.completed` |
| `sale.completed` | `sale.completed` |
| `order.placed` | `sale.completed` |
| `order.refund` | `refund.created` |
| `refund.issued` | `refund.created` |
| `order.void` | `sale.voided` |
| `order.cancelled` | `sale.voided` |
| `order.canceled` | `sale.voided` |

**Source:** `providers/restrox/mapper.js` — `EVENT_TYPE_MAP`.

---

### 7. Unknown Event Type Fallback

**Gap:** Not documented anywhere. Unknown event types silently map to `sale.completed`. A sender using an unrecognized event type gets `200 Event received` but the event is processed as a completed sale.

**Source:** `providers/restrox/mapper.js` line 43 — `EVENT_TYPE_MAP[parsed.event_type] || "sale.completed"`.

---

### 8. Idempotency Key Construction

**Gap:** Docs said "the transaction ID is used for duplicate detection." This was incomplete.

**Actual behavior:**
- Key format: `{event_type}:{order_id}`
- A sale and refund for the same `order_id` are **not** duplicates (different key prefix)
- If no order ID exists, SHA-256 fallback from `{provider}:{event_type}:{order_id}:{amount}:{timestamp}`
- Scope: per-integration (same `order_id` across two merchants does not conflict)

**Source:** `controller.js` — `extractIdempotencyKey` function.

---

### 9. All Blocked Reason Codes

**Gap:** `response-reference.md` documented `blocked_unmapped_location` but didn't explain the distinct reasons that lead to a block. Troubleshooting was incomplete for integrators whose events arrive but no loyalty is awarded.

**Blocked reasons added to `response-reference.md` and `troubleshooting.md`:**

| Blocked Reason | Cause |
| -------------- | ----- |
| `missing_external_location_id` | Payload had no location field |
| `unmapped_location` | Location field present but no matching location record found |
| `stale_location_mapping` | Matched location has `location_status === "STALE"` (outlet removed) |
| `participation_disabled` | Location has `participation_enabled === false` |
| `inactive_location_mapping` | Location record `status !== "active"` and no outlet linked |

**Source:** `locationResolutionService.js` — `getBlockedReason` function.

---

### 10. Phone Normalization Country Default

**Gap:** `customer-identity.mdx` did not document that phone numbers without a country prefix are assumed to be Nepal (`NP`).

**Source:** `providers/restrox/mapper.js` — `PROVIDER_DEFAULT_COUNTRY = "NP"`.

---

### 11. `CONNECTED` Status Is Unreachable

**Gap:** `merchant-onboarding.mdx` listed `CONNECTED` among the verified overall status values while also noting it is never emitted. This was contradictory. The status exists as a constant but the health builder never assigns it.

**Fix:** Removed `CONNECTED` from the list of returned states. Documented it as a constant that is not emitted in the current implementation.

**Source:** `merchant/service.js` — `buildIntegrationHealth` function.

---

### 12. `503` Error for Unconfigured Partner Key

**Gap:** `partner-api.mdx` error conditions did not list the `503` response that occurs when the `RESTROX_PARTNER_API_KEY` environment variable is not configured.

**Source:** `partners/restrox/middleware.js` lines 6–9.

---

### 13. Refund Identifier Matching Requirement

**Gap:** `refunds.md` mentioned the original sale identifier but didn't clarify that the same field name must be used and that the idempotency key prefix distinguishes the refund from the original sale.

**Source:** `controller.js` — `extractIdempotencyKey`; `locationResolutionService.js` — `getBlockedReason`.

---

### 14. Integration Checklist Missing Native Onboarding Items

**Gap:** The checklist only covered webhook delivery. It had no steps for the native connect flow, location sync, or customer lookup that precede webhook traffic.

---

## Integration Requirements Discovered During Implementation

The following behaviors were established during implementation and testing but were not previously documented:

1. **Token-based webhook routing is the only active production path.** The legacy `POST /integrations/pos/:provider/events` route returns `410 Gone` unless `POS_LEGACY_PROVIDER_ROUTE_ENABLED=true` is set. Partners must use per-location webhook tokens, not API keys.

2. **Webhook tokens are per-location, not per-integration.** Each `PosIntegrationLocation` record has its own `webhook_token`. The merchant integration panel exposes the URL per location.

3. **`external_location_id` in the webhook payload is matched against the value stored during location sync.** The value must match exactly (string comparison). Case and spacing matter.

4. **Location auto-matching during sync uses normalized name comparison** (lowercase, alphanumeric only). A name mismatch that would be obvious to a human may still fail auto-match if punctuation or spacing differs.

5. **Draft location rows exist with placeholder `external_location_id` values** (`__draft__:{outletId}`) before partner sync completes. These draft values are never valid event location identifiers.

6. **`integration_key_last_used_at` is updated after every successful test-sale call**, not just after connect. This is a useful signal for debugging whether test sales are reaching the integration.

7. **Integration rotation resets the entire connection state.** After rotation: `partner_connection_status` → `pending`, `partner_connected_at` → null, `partner_metadata` → empty, `status` → `inactive`. RestroX must reconnect from the beginning.

8. **Customer search query parameters beyond `phone` are rejected.** The `search` endpoint only accepts `phone`. Any additional query parameters cause a `400 Search supports exact phone lookup only` error.

9. **Store-scoped customer isolation.** The `x-integration-key` in customer API requests resolves the store, and customer lookup is bounded to that store. The same phone number could belong to customers in different stores without conflict.

10. **All inbound webhook requests produce a raw audit record**, even before authentication. This means failed, duplicate, and unauthorized requests are all observable in the internal event log.
