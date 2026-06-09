---
title: Endpoint Catalog
description: Endpoint inventory for the RestroX native integration and webhook-backed event delivery flow.
sidebarTitle: Endpoint Catalog
---

# Endpoint Catalog

This catalog lists the externally relevant RestroX routes documented in this package.

## Native Partner APIs

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/api/partners/restrox/connect` | Connect a merchant using a Connection Key and optionally sync locations | Partner key | Active |
| `POST` | `/api/partners/restrox/sync-locations` | Sync RestroX locations into the merchant integration | Partner key | Active |
| `POST` | `/api/partners/restrox/test-sale` | Submit a test sale through the existing event pipeline | Partner key | Active |

## Partner Customer APIs

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `GET` | `/api/partners/{provider}/customers/search` | Search by exact phone within the merchant store | Partner key + Connection Key | Active |
| `POST` | `/api/partners/{provider}/customers` | Create a customer or return an existing match | Partner key + Connection Key | Active |
| `POST` | `/api/partners/{provider}/customers/upsert` | Create-or-return-existing by normalized phone | Partner key + Connection Key | Active |
| `GET` | `/api/partners/{provider}/customers/{customerId}` | Fetch one customer by ID | Partner key + Connection Key | Active |
| `PATCH` | `/api/partners/{provider}/customers/{customerId}` | Update name or email | Partner key + Connection Key | Active |

## Webhook And Event Transport

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/webhook/restrox/{token}` | Receive RestroX sale, refund, and void webhooks | Token in URL path | Active |
| `POST` | `/integrations/pos/{provider}/events` | Provider event ingestion route used by the native test-sale path | `x-api-key` | Active |
