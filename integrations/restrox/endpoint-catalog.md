---
title: Endpoint Catalog
description: Endpoint inventory for the RestroX native integration and webhook-backed event delivery flow.
sidebarTitle: Endpoint Catalog
---

This catalog lists the externally relevant RestroX routes documented in this package.

## Native Partner APIs

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/api/partners/restrox/connect` | Connect a merchant using a Connection Key and optionally sync locations | Partner key | Active |
| `POST` | `/api/partners/restrox/sync-locations` | Sync RestroX locations into the merchant integration | Partner key | Active |
| `POST` | `/api/partners/restrox/test-sale` | Submit a test sale that validates mapped location behavior and loyalty processing readiness | Partner key | Active |

## Partner Customer APIs

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `GET` | `/api/partners/{provider}/customers/search` | Search for a Samparka-owned customer by exact phone within the merchant store | Partner key + Connection Key | Active |
| `GET` | `/api/partners/{provider}/customers/{customerId}` | Retrieve one Samparka-owned customer by ID | Partner key + Connection Key | Active |

Customers are owned by Samparka. Partners may search and retrieve customers only. Partner customer APIs are read-only.

## Webhook And Event Transport

| Method | Path | Purpose | Auth | Status |
| ------ | ---- | ------- | ---- | ------ |
| `POST` | `/webhook/restrox/{token}` | Receive RestroX sale, refund, and void webhooks | Token in URL path | Active |
