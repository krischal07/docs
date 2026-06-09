# POS Integration API Audit

## Existing Endpoint Inventory
- `POST /webhook/restrox/:token`
- `POST /integrations/pos/:provider/events`
- `GET /admin/pos-location-mappings`
- `POST /admin/pos-location-mappings`
- `PUT /admin/pos-location-mappings/:id`
- `GET /admin/raw-webhook-events`
- `GET /admin/raw-webhook-events/:id`
- `GET /admin/internal-events`
- `GET /admin/internal-events/:id`
- `POST /admin/internal-events/:id/replay`
- `POST /admin/internal-events/:id/reprocess`

## Missing Merchant Self-Service Inventory
These were not present before implementation and required a merchant-facing layer:
- `GET /api/pos-integrations/providers`
- `GET /api/pos-integrations`
- `POST /api/pos-integrations`
- `GET /api/pos-integrations/:id`
- `PATCH /api/pos-integrations/:id`
- `GET /api/pos-integrations/:id/locations`
- `PUT /api/pos-integrations/:id/locations`
- `POST /api/pos-integrations/:id/locations/:locationId/regenerate-token`
- `POST /api/pos-integrations/:id/sync-outlets`
- `GET /api/pos-integrations/:id/activity`
- `GET /api/pos-integrations/:id/status`
- `POST /api/pos-integrations/:id/verify`
- `GET /api/pos-integrations/:id/onboarding`
- `GET /api/pos-integrations/:id/locations/:locationId/webhook-url`

## Existing Models Reused
- `PosIntegration`
- `PosIntegrationLocation`
- `RawWebhookEvent`
- `InternalEvent`
- `ProcessedEvent`
- `Store`
- `Outlet`

## Existing Controllers Reused
- POS webhook ingestion controller
- POS admin observability controllers

## Existing Services Reused
- POS location resolution service
- POS event processing pipeline
- POS admin query services

## New Services Required
- Merchant POS integration service
- Provider metadata source
- Onboarding/status calculator
- Merchant ownership checks
- Merchant audit trail writer

## Notes
- The backend source of truth remains per-location webhook routing.
- Merchant APIs are layered on top of existing webhook processing, not a separate POS subsystem.
- The self-service flow required draft location rows before outlet mappings exist, so `PosIntegrationLocation` now supports null `external_location_id` values until merchants complete mapping.
