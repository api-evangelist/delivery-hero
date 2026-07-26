---
name: Create and track an On Demand Rider delivery
description: Authenticate, register an outlet, estimate fees/time, create an on-demand delivery order, and track it to completion using the Delivery Hero On Demand Rider API.
api: openapi/delivery-hero-on-demand-rider-openapi.json
operations:
  - POST /oauth2/token
  - PUT /outlets/{client_vendor_id}
  - POST /orders/fee
  - POST /orders/time
  - POST /orders
  - GET /orders/{order_id}
  - GET /orders/{order_id}/coordinates
---

# Create and track an On Demand Rider delivery

Use the Delivery Hero On Demand Rider (ODR) API to dispatch an on-demand courier.

## Auth
1. Obtain a token: `POST /oauth2/token` against `https://sts.deliveryhero.io/oauth2/token`,
   grant type `client_credentials`, using a signed JWT assertion
   (`urn:ietf:params:oauth:client-assertion-type:jwt-bearer`). Your ClientID, KeyID and
   Scope (`{brand}.api.{country_code}.*`) are issued by ODR after you register an RSA-2048 public key.
2. Send `Authorization: Bearer {access-token}` on every subsequent request.

## Steps
1. **Register the pickup outlet** — `PUT /outlets/{client_vendor_id}` to create or update the
   store the delivery is dispatched from.
2. **Estimate** — `POST /orders/fee` and `POST /orders/time` to quote delivery fee and ETA
   before committing.
3. **Create the order** — `POST /orders` with pickup outlet, recipient contact/address, and
   package details. Capture the returned `order_id`.
4. **Track** — poll `GET /orders/{order_id}` for status and `GET /orders/{order_id}/coordinates`
   for live rider position, or (preferred) consume the `order.status` and `courier.location`
   webhooks (see asyncapi/delivery-hero-on-demand-rider-webhooks.yml).

## Rules
- Base URL is brand- and country-specific: `https://{brand}-api-{region}.deliveryhero.io/{country_code}/api/v1/`.
- No idempotency-key header is documented — avoid blind retries of `POST /orders`; check status first.
- Errors return `{ "errors": [ ... ] }` (see errors/delivery-hero-problem-types.yml), not RFC 9457.
- Verify inbound webhooks with the `X-Signature-SHA256` header (HMAC-SHA256).
