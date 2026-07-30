---
name: Quote and execute a currency exchange or internal transfer
description: Get an FX quote and move funds between two accounts of different currencies, or transfer between same-currency accounts, on Lorum (Fuse).
api: openapi/lorum-openapi-original.json
operations: [get_exchange_quote, create_exchange, create_internal_transfer, list_all_customer_accounts, get_account_history]
---

# Quote and execute a currency exchange or internal transfer

## Auth
OAuth2 client-credentials bearer token (cached, 3-hour lifetime).

## Cross-currency exchange
1. Discover accounts with `list_all_customer_accounts` to find the source and destination
   `account_id`s (they must hold different currencies for an exchange).
2. Get a rate with `get_exchange_quote` (from/to currency).
3. Execute with `create_exchange` (`from_account_id`, `to_account_id`), sending a
   `Idempotency-Key` header. A `404` here can mean no rate is available for the pair; a `400`
   means the pair is unsupported.

## Same-currency transfer
- Use `create_internal_transfer` to move funds between two same-currency accounts
  (`source_account_id`, `counterpart_account_id`), again with an `Idempotency-Key`.

## Verify
- Check balances/movements with `get_account_history`.
- React to the Exchange/Transfer webhook events
  (`outbound_exchange_transfer_executed`, `inbound_exchange_transfer_settled`,
  `internal_transfer_settled`, `internal_transfer_failed`).

## Rules
- Amounts are integer minor units; currencies are ISO 4217.
- Send a fresh UUIDv4 `Idempotency-Key` per distinct move.
