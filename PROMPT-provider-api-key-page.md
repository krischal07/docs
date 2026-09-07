# PROMPT — Superadmin: Provider API Key Generation Page

> Copy this entire file into the agent working in the **samparka-backend** repo (the project where the System integration endpoints live, e.g. `src/integrations/system/`). All file paths mentioned below describe the backend as documented; verify against the actual codebase before coding.

## Scope (current phase)

**Focus only on the two connected POS providers: RestroX and Blanxer.**

- Both are already registered as `Provider` records (seeded via `bootstrapProviders.js`) and both already have full adapters (`validate` / `parser.parse` / `mapper.map`) in `src/integrations/system/providers/`.
- No provider registration or adapter work is needed in this task — only key generation and management.
- The page should list whatever providers exist, but the initial rollout issues keys only for `restrox` and `blanxer`.
- Do not build vendor self-signup or a provider-creation flow in this phase.

## Context

Samparka has a provider-agnostic System integration. Every POS vendor (provider) calls endpoints like:

- `POST /integrations/system/{provider}/purchase-qr`
- `POST /api/partners/{provider}/connect`
- `POST /api/partners/{provider}/test-sale`

Auth on these endpoints is two-part:

1. **Provider API key** — `Authorization: Bearer spkp_...`. Identifies the POS **vendor** (not the store). The key's `slug` must match the `{provider}` path segment. Validated by the existing `authenticateProviderApiKey` middleware. The same vendor key is shared by ALL stores running that POS.
2. **Integration key** — `X-Integration-Key` (or `x-integration-key` header on customer APIs). Identifies the **store/outlet**. Different for every store. This is a separate system — do not touch it in this task.

Provider and key data already exist in the backend:

- `Provider` model — seeded via `bootstrapProviders.js` (currently Samparka, Blanxer, RestroX)
- `ProviderApiKey` model — stores partner credentials (keys have a `slug` that must match the provider)
- Admin API surface — provider administration is mounted under `/api/admin/system/providers/*` (provider catalogue, provider keys, provider integrations/activity)

## Goal

Build a **superadmin-only page** in the Samparka dashboard to **generate and manage provider API keys**, so that onboarding a store with RestroX or Blanxer is: pick provider (`restrox` / `blanxer`) → generate key → hand the key to the POS vendor → vendor pastes it into their POS settings alongside the store's integration key.

## Requirements

### 1. Access control (must)

- Page and all its API routes restricted to the **superadmin** role only. Use the existing admin auth middleware pattern; add an explicit superadmin check — other admin roles must get `403`.
- Every action must be audit-logged: who created/revoked/rotated which key for which provider, when.

### 2. Page — List view

For each provider (show `restrox` and `blanxer` first):

- Provider name, slug, active/capable status
- Its API keys in a table: label, key prefix (e.g. `spkp_abc1...` — never the full key), status (active/revoked), created date, created by, last used (if tracked), expiry (if set)
- Actions per key: **Revoke**, **Rotate** (revoke + generate replacement in one step)

### 3. Page — Generate key (the core feature)

- Button: **"Generate API Key"** on each provider row, or a global one where the admin first picks the provider (dropdown limited to `restrox` and `blanxer` for now).
- Optional inputs: a **label** (e.g. "RestroX production key"), optional **expiry date**.
- On submit, backend:
  - Generates a cryptographically random key with the existing **`spkp_` prefix** format (reuse whatever generator `authenticateProviderApiKey`/key seeding already uses — do not invent a new format).
  - Stores it the same way existing keys are stored (hashed if that's the current pattern — check how existing seeded keys are persisted).
  - Binds the key to the provider `slug` (`restrox` or `blanxer`).
- **Show the full key exactly once** in the response UI, with a copy button and the warning: *"You won't be able to see this key again. Copy it now."* After dismissal, only the masked prefix is ever shown.

### 4. API endpoints (add if missing, reuse if present)

Check `/api/admin/system/providers/*` first — if key management routes already exist, extend rather than duplicate. Expected shape:

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/api/admin/system/providers` | List providers (for the picker) |
| `GET` | `/api/admin/system/providers/:slug/keys` | List keys for a provider (masked) |
| `POST` | `/api/admin/system/providers/:slug/keys` | Generate a new key; response returns the **full key once** |
| `DELETE` | `/api/admin/system/providers/:slug/keys/:keyId` | Revoke a key |
| `POST` | `/api/admin/system/providers/:slug/keys/:keyId/rotate` | Revoke + issue replacement |

- All routes behind superadmin middleware.
- Reject key creation for any slug other than the registered providers (for now effectively `restrox` and `blanxer`).
- Revoking a key must immediately invalidate it for partner calls (`401` on next use) — confirm `authenticateProviderApiKey` checks key status on every request, not just existence.
- Revoking/rotating must not touch the store-level integration keys (`X-Integration-Key`) — those are a separate system.

### 5. UX details

- Clear separation on the page between **provider API keys** (vendor identity, shared by all stores of that POS) and **integration keys** (store identity, unique per store) — link to docs explaining the two-key model so admins don't confuse them.
- After generating, show a "handoff" hint: the POS vendor will call `POST /integrations/system/{slug}/purchase-qr` with `Authorization: Bearer <key>` (the store is resolved from the key's `store_id` scope), and partner routes like `POST /api/partners/{slug}/connect` the same way.
- Confirmation dialog before revoke/rotate (revoke breaks the vendor's live calls at every store using that key).

### 6. Acceptance criteria

- [ ] Superadmin can log in, open the page, and generate a key for `restrox` and for `blanxer`.
- [ ] Non-superadmin admin gets `403` on both the page and all key API routes.
- [ ] Generated key works immediately: a curl to `POST /integrations/system/restrox/purchase-qr` with the new Bearer key + a connected store's integration key returns `200` with `qr_link` (same for `blanxer`).
- [ ] A key whose slug does not match the URL `{provider}` gets rejected (`401`/`404`) — existing behavior must not regress.
- [ ] Revoked key gets `401` on the next partner call.
- [ ] Full key is displayed once only; list views show masked prefixes.
- [ ] Audit log entries exist for generate/revoke/rotate.

### 7. Out of scope

- Do not change the purchase-qr, partner, or webhook endpoint contracts.
- Do not touch store/outlet integration key issuance — that stays where it is.
- No self-service vendor signup; keys are issued by superadmin only.
- No new provider registration / adapter development (RestroX and Blanxer adapters already exist).

## Deliverables

1. Backend routes + superadmin middleware guard + audit logging.
2. Dashboard page (list + generate + revoke/rotate UI, providers limited to `restrox` / `blanxer` in the picker).
3. Short note in the integration docs (`integrations/system/`) describing where admins generate keys, so the partner docs can reference it.
