---
name: Estimate and create an outbound payment
description: Estimate the fee for an outbound local payment and create it safely with an idempotency key, then track its transaction on Lorum (Fuse).
api: openapi/lorum-openapi-original.json
operations: [customer_payment_fee_estimate, create_customer_payment, list_customer_transactions, get_transaction, get_transaction_confirmation_letter]
---

# Estimate and create an outbound payment

## Auth
OAuth2 client-credentials bearer token (see the onboarding skill). Reuse the cached 3-hour token.

## Steps
1. (Optional) Estimate the fee with `customer_payment_fee_estimate` before committing.
2. Create the payment with `create_customer_payment`. Send a **required** `Idempotency-Key`
   header (UUIDv4). Amounts are integers in minor units (100 = 1.00); currency is ISO 4217.
3. The call returns a transaction reference. Track it with `get_transaction`, or list an
   account's activity with `list_customer_transactions`.
4. For `submitted` or `executed` outbound transactions you can fetch a
   `get_transaction_confirmation_letter`.

## Outcome handling
Payment settlement is asynchronous. Do not treat the synchronous response as final — subscribe
to the Payment webhook category and react to `outbound_local_payment_submitted`,
`outbound_local_payment_executed`, `outbound_local_payment_failed`, and
`outbound_local_payment_returned`. Deduplicate on `event_id`.

## Rules
- Never mint a fresh token per call; cache it (mandatory for production certification).
- Always send a fresh UUIDv4 `Idempotency-Key` per distinct payment; retry with the same key.
