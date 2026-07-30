---
name: Simulate payment scheme actions in the sandbox
description: Drive a payment end-to-end in the Lorum (Fuse) sandbox by simulating inbound settlement, outbound execution, failure, and returns.
api: openapi/lorum-openapi-original.json
operations: [create_customer_payment, simulate_inbound_payment, simulate_execute_payment, simulate_fail_payment, simulate_return_payment]
---

# Simulate payment scheme actions in the sandbox

The sandbox stands in for the real payment scheme so you can test full flows without waiting
for settlement. **All simulate functions are sandbox-only** (`https://api-sandbox.fuse.me`).

## Steps
1. Fund/receive: `simulate_inbound_payment` to simulate money settling into an account.
2. Create an outbound payment with `create_customer_payment` (with `Idempotency-Key`).
3. Advance it: `simulate_execute_payment` to simulate the scheme executing a submitted
   outbound payment.
4. Or exercise the unhappy paths: `simulate_fail_payment` (submitted payment fails) and
   `simulate_return_payment` (executed payment returned by the beneficiary's bank).
5. Verify each transition arrives on your webhook endpoint
   (`inbound_local_payment_settled`, `outbound_local_payment_executed`,
   `outbound_local_payment_failed`, `outbound_local_payment_returned`).

## Rules
- These operations exist only in sandbox; do not build production logic that calls them.
- Completing the integration checklist (token caching, idempotency, webhook handling) is
  required before production API access is granted.
