---
name: Onboard a customer and open an account
description: Onboard a business or individual customer for a currency, then open a multi-currency account for them on Lorum (Fuse).
api: openapi/lorum-openapi-original.json
operations: [add_business, add_individual, onboard_customer, get_customer_onboarding, create_customer_account, get_customer_account]
---

# Onboard a customer and open an account

Use this to bring a new customer onto Lorum and give them a usable account.

## Auth
Mint an OAuth2 client-credentials bearer token from `https://auth-sandbox.fuse.me/oauth/token`
(sandbox) or `https://auth.fuse.me/oauth/token` (production), audience `https://api.fuse.me`.
Cache and reuse it for its full 3-hour lifetime; send `Authorization: Bearer <token>` on every call.

## Steps
1. Create the customer record: `add_business` (for a company) or `add_individual` (for a person).
   Capture the returned `customer_id`.
2. Onboard the customer for the target currency with `onboard_customer` (path `customer_id`,
   currency in the body). Onboarding is per-currency (AED, USD, EUR, GBP, ...).
3. Poll `get_customer_onboarding` until the currency reaches a completed status
   (or handle the `onboarding_completed` / `onboarding_failed` webhook instead of polling).
4. Open the account with `create_customer_account` — send a **required** `Idempotency-Key`
   header (UUIDv4), plus `account_name`, `external_id`, `currency`, and `account_type: virtual`
   (`payment` is deprecated). This returns `202 Accepted` with an `account_id`.
5. Confirm with `get_customer_account`.

## Rules
- `Idempotency-Key` is mandatory on account creation; reuse the same key to safely retry,
  a `409` means the original is still in flight, a `422` means you reused the key with a
  different payload.
- Account opening is asynchronous (`202`); wait for the `account_opened` webhook.
