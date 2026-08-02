---
name: Sync catalog and inventory in Vin eRetail
description: Create/update SKUs and push on-hand inventory levels to Vin eRetail so channels stay in stock.
api: openapi/vinculum-eretail-openapi-original.yml
operations:
  - POST /RestWS/api/eretail/v1/sku/create
  - POST /RestWS/api/eretail/v1/sku/update
  - POST /RestWS/api/eretail/v4/stock/updateInventory
---

# Sync catalog and inventory in Vin eRetail

Use this skill to keep product master data and inventory current across Vinculum-managed channels.

## Authentication
Send the `ApiOwner` + `ApiKey` credential pair (form fields / headers / query) on every call, plus
`DbId` on multi-tenant deployments. See `authentication/vinculum-authentication.yml`.

## Steps
1. **Create products** — `POST /RestWS/api/eretail/v1/sku/create` (max 100 lines/hit). Send SKU
   master records in `RequestBody`.
2. **Update products** — `POST /RestWS/api/eretail/v1/sku/update` for changes to existing SKUs
   (max 100 lines/hit).
3. **Push inventory** — `POST /RestWS/api/eretail/v4/stock/updateInventory` with SKU + location +
   quantity (max 100 lines/hit). This is the operation that keeps the 150+ connected sales
   channels in stock.

## Rules
- Batch within the documented line limits per call and stay under the 5-minute quota
  (`rate-limits/vinculum-rate-limits.yml`).
- Inspect `responseCode`/`responseMessage` and the per-record `requestStatus` — partial batches
  can succeed while individual lines fail (`errors/vinculum-error-codes.yml`).
- Inventory is keyed by SKU + `orderLocation`; see `data-model/vinculum-data-model.yml`.
