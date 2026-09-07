# PROMPT — Connect My POS to Samparka Loyalty (Purchase QR on Receipt)

> Use this file as the brief for building the **POS-side integration** that prints a Samparka loyalty QR on cash-sale receipts.

## What We're Building

When a cashier completes a **cash sale**, the POS calls Samparka's Purchase QR endpoint, receives a `qr_link`, renders it as a QR image, and prints it on the **normal receipt**. The customer scans it → WhatsApp claim flow → loyalty points awarded. The POS is the **client** here; the endpoint already runs on Samparka's servers — no server work on the POS side.

```txt
Cash sale completed
  → POST {baseUrl}/integrations/system/{provider}/purchase-qr
  ← { qr_link, ... }
  → render qr_link as QR image
  → print on the normal receipt
```

## 1. Settings (one-time config)

Add a Samparka section to POS settings with:

| Field | Example | Notes |
| --- | --- | --- |
| Base URL | `https://server.samparka.co` | `http://localhost:8081` for local dev against the backend repo |
| Provider API key | `spkp_...` | Vendor identity. Issued by Samparka (superadmin page). Shared across all stores running this POS. |
| Integration key | store integration key | Store identity. Unique per store. Provided during onboarding. |
| `{provider}` slug | e.g. `restrox`, `blanxer`, or this POS's own slug | Must match the provider the API key was issued for; used in the URL path |

Persist these; a "Test connection" button that fires a validation-only request is a nice-to-have.

## 2. The API Call (at cash-sale completion)

```http
POST {baseUrl}/integrations/system/{provider}/purchase-qr
Content-Type: application/json
Authorization: Bearer <provider_api_key>
```

```json
{
  "amount": 1250,
  "currency": "NPR",
  "items": [
    { "name": "Cappuccino", "qty": 1, "price": 850 }
  ]
}
```

Field rules:

- `amount` — **required**, number, must be `> 0`
- `currency` — optional, 3-letter code, defaults to `NPR` if omitted
- `items` — **required**, array of `{ name, qty, price }`; `qty` and `price` must be positive; **the items' total must equal `amount`** or the request is rejected with `400`

Success — HTTP `200`:

```json
{
  "success": true,
  "message": "Purchase QR ready",
  "data": {
    "qr_link": "https://samparka.co/r/xKd93k",
    "purchase_reference": "ps_1690000000000_ab12cd34ef56",
    "amount": 1250,
    "currency": "NPR",
    "status": "QR_GENERATED",
    "expires_at": "2026-08-26T12:12:04.000Z",
    "integration_key": "xxxxxxxxxxxxxxxx"
  }
}
```

`data.qr_link` is the short URL to render as the QR.

## 3. QR Rendering & Printing

- Take `data.qr_link` and render it as a QR image using any QR library available on the POS platform (e.g. `qrcode` for JS, platform-native QR APIs, or the receipt printer's built-in QR support for ESC/POS).
- Print the QR on the normal receipt (below or beside the bill totals). Optionally add a caption like "Scan to earn loyalty points".
- Printing continues to work normally even if QR generation fails — the sale must never be blocked by loyalty (see error handling).

## 4. Error Handling (never block the sale)

| HTTP | Meaning | POS behavior |
| --- | --- | --- |
| `400 purchase_qr_validation_failed` | Bad body — amount/items/currency rules violated | Log it; this is a POS bug, fix the payload. Print receipt without QR. |
| `401` | Invalid/missing provider API key or integration key | Surface a config error in settings; cashier sees "Loyalty not configured". Print receipt without QR. |
| `404 unknown provider` | `{provider}` slug wrong | Same as 401 — config problem. |
| `409 system_not_connected` | Store integration is not `CONNECTED`/`ACTIVE` | Config/onboarding issue; notify store admin. Print receipt without QR. |
| `422 no active connected communication provider` | Store has no active WhatsApp/Wapio provider | Same — store-side setup issue. Print receipt without QR. |
| `409 already processed` | The session was already completed/claimed/failed | Do **not** retry. Print receipt without QR. |
| `410 expired` | The session expired | Do **not** retry. Print receipt without QR. |
| `502 QR generation failed` | Samparka-side QR/redirect failure | Safe to retry the same request once. If it fails again, print without QR. |
| Network timeout / no response | Unknown outcome | Safe to retry: see retry rules below. |

**Golden rule:** loyalty QR failure must never block or fail the sale. Always print the receipt.

## 5. Retry & Idempotency Rules

- **Each successful request creates a new session** (`ps_…` reference). There is no cross-request dedup keyed on a bill ID.
- **Retrying while the session is still pending is safe:** if the previous request reached Samparka and the session is `QR_GENERATED` / `WAITING_FOR_CUSTOMER_CLAIM`, a retry of the *same session* returns the **same `qr_link`** with `200`.
- Practical pattern: on timeout/unknown outcome, retry the identical request body once after a short delay. If you get a `qr_link`, print it.
- If retrying creates a second session (i.e., the first request never landed), the worst case is an unprinted QR that expires — harmless.
- `409 already processed` / `410 expired` are **terminal** — stop retrying, move on.

## 6. Environments

| Environment | Base URL | Notes |
| --- | --- | --- |
| Local dev | `http://localhost:8081` | Run the samparka-backend locally. Good for payload/error testing. |
| Staging | staging server URL | Test the full scan flow with real phones. |
| Production | `https://server.samparka.co` | Live. QR links resolve as `https://samparka.co/r/...`. |

⚠️ **Localhost gotcha:** QR links generated against localhost (`http://localhost:8081/r/...`) are **not reachable from a customer's phone**. You can verify the API contract locally, but the real scan → WhatsApp → points flow must be tested on staging/production where short URLs are public.

## 7. Prerequisites Checklist (before coding against prod)

- [ ] Provider API key (`spkp_…`) issued by Samparka superadmin for this POS's slug
- [ ] Store integration key in hand, and the store integration is `CONNECTED`/`ACTIVE` (not `CREATED`)
- [ ] An active WhatsApp/Wapio communication provider configured for the store
- [ ] `{provider}` slug confirmed and matching the key

## 8. Out of Scope (for this task)

- **Webhook events** (`order.completed`, refunds, voids) — that's a separate Samparka-side adapter; don't implement sending these from the POS in this task.
- Customer lookup APIs — merchant/partner side, not the POS terminal.
- Offline-first queueing — if the terminal is offline at sale time, the simple v1 behavior is: print receipt without QR (optionally queue a retry; decide if needed).

## Acceptance Criteria

- [ ] Settings screen saves and persists base URL, API key, integration key, provider slug
- [ ] Completing a cash sale triggers the purchase-qr call automatically
- [ ] On success, the QR (rendered from `qr_link`) appears on the printed receipt
- [ ] Invalid/missing keys produce a clear settings/config error — the sale still completes and prints
- [ ] Network failure mid-request → one safe retry; terminal errors (`409 already processed`, `410 expired`) are not retried
- [ ] Sale is never blocked or failed because of loyalty/QR errors
- [ ] Full flow verified once on staging with a real phone scan: print → scan → WhatsApp claim → points awarded
