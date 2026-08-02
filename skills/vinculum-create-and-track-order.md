---
name: Create and track an order in Vin eRetail
description: Create an order, poll its processing status, and retrieve its current fulfillment status using the Vin eRetail API.
api: openapi/vinculum-eretail-openapi-original.yml
operations:
  - POST /RestWS/api/eretail/v4/order/create
  - POST /RestWS/api/eretail/v1/common/checkRequestStatus
  - POST /RestWS/api/eretail/v4/order/status
---

# Create and track an order in Vin eRetail

Use this skill to push an order into Vin eRetail and follow it to a confirmed status.

## Authentication
Every call requires the `ApiOwner` + `ApiKey` credential pair issued by Vinculum. Send them as
form fields (the write operations consume `application/x-www-form-urlencoded` with the JSON
payload in the `RequestBody` field), or as headers/query params. On multi-tenant deployments also
send the `DbId` header. See `authentication/vinculum-authentication.yml`.

## Steps
1. **Create the order** — `POST /RestWS/api/eretail/v4/order/create`. Send `ApiOwner`, `ApiKey`,
   and the order JSON in `RequestBody`. Quota: 80 calls / 5 min. The response envelope returns
   `responseCode` + `responseMessage`; a nested `requestStatus` carries per-line acceptance.
2. **Confirm async processing** — if the order is queued, poll
   `POST /RestWS/api/eretail/v1/common/checkRequestStatus` with the returned request reference
   until it reports completion. (There is no client idempotency key; this poll is the retry-safe
   confirmation path.)
3. **Read fulfillment status** — `POST /RestWS/api/eretail/v4/order/status` with the order number
   to get the current status. For listing/filtering multiple orders use
   `POST /RestWS/api/eretail/v3/order/orderPull` (page through with `pageNo`; read `totalPages`).

## Rules
- Check `responseCode`/`responseMessage` on every response — the API returns HTTP 200 even for
  handled application errors (see `errors/vinculum-error-codes.yml`).
- Respect the per-operation 5-minute quotas (`rate-limits/vinculum-rate-limits.yml`).
