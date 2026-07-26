---
name: Cancel an order and retrieve delivery proofs
description: Cancel or update an in-flight On Demand Rider order and fetch proof of pickup, delivery, and return using the Delivery Hero On Demand Rider API.
api: openapi/delivery-hero-on-demand-rider-openapi.json
operations:
  - POST /oauth2/token
  - GET /orders/{order_id}
  - PUT /orders/{order_id}
  - DELETE /orders/{order_id}
  - GET /orders/proof_of_pickup/{order_id}
  - GET /orders/proof_of_delivery/{order_id}
  - GET /orders/proof_of_return/{order_id}
---

# Cancel an order and retrieve delivery proofs

## Auth
Obtain a Bearer token via `POST /oauth2/token` (client_credentials + JWT bearer) and send
`Authorization: Bearer {access-token}`.

## Steps
1. **Check current state** — `GET /orders/{order_id}` before mutating.
2. **Update** — `PUT /orders/{order_id}` to modify a still-mutable order.
3. **Cancel** — `DELETE /orders/{order_id}`. Orders past a certain lifecycle point are
   uncancellable and return HTTP 409 (Conflict / Uncancellable).
4. **Collect proofs** once delivered/returned:
   - `GET /orders/proof_of_pickup/{order_id}`
   - `GET /orders/proof_of_delivery/{order_id}`
   - `GET /orders/proof_of_return/{order_id}`

## Rules
- A 409 on cancel is expected and final — do not retry; reconcile via the `order.status` webhook.
- Refunds arrive asynchronously via the `order.refund` webhook (`POST /callback-refund`).
- Errors use the `{ "errors": [ ... ] }` envelope; 404 means the order/outlet was not found.
