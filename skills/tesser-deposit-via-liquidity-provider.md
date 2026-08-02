---
name: tesser-deposit-via-liquidity-provider
description: Use when a developer wants to deposit fiat funds into their managed Tesser accounts via a liquidity provider (on/off-ramp) — register a bank account, create the deposit, fetch wire/instructions, and track it to settlement.
api: openapi/tesser-openapi-original.json
method: generated
operations: [account_createBankAccount, treasury_createDeposit, treasury_getDepositInstructions, treasury_getDeposit, treasury_simulateDeposit]
---

# Deposit fiat via a liquidity provider on Tesser

Deposits on-ramp fiat into your Tesser-managed accounts (optionally converting to
stablecoin). Funds settle through a connected liquidity provider (e.g. OpenFX).

## Prerequisites

- Access token (client-credentials JWT, `audience=https://api.tesser.xyz`).
- A connected liquidity provider. To connect OpenFX, use the provider-published
  `setup-openfx` skill (skills/tesser-setup-openfx.md).

## Steps

1. **Register the source bank account** — `account_createBankAccount`
   (`POST /v1/accounts/banks`) with the bank-account fields from the object
   reference.
2. **Create the deposit** — `treasury_createDeposit` (`POST /v1/treasury/deposits`)
   specifying the amount, currency, destination account, and whether to on-ramp to
   stablecoin.
3. **Get instructions** — `treasury_getDepositInstructions`
   (`GET /v1/treasury/deposits/{id}/instructions`) to obtain the wire / funding
   instructions to send fiat to.
4. **Track it** — `treasury_getDeposit` (`GET /v1/treasury/deposits/{id}`) until it
   reaches a terminal status, or subscribe to `deposit.*` webhooks.
5. **Sandbox only:** `treasury_simulateDeposit`
   (`POST /v1/treasury/deposits/{id}/simulate`) simulates the inbound funds so you
   can exercise the full lifecycle without moving real money.

## Rules

- **Auth / pagination / errors:** identical to the payout skill — bearer JWT,
  `page`+`limit`, `{ errors: [ { error_code, error_message } ] }`.
- **Environments:** sandbox is `https://sandbox.tesserx.co`; production is
  `https://api.tesser.xyz`. Simulation endpoints work in sandbox/staging.
- Fetch the live OpenAPI at `https://docs.tesser.xyz/api/v1/schema.json` to confirm
  the current bank-account and deposit payload fields before writing.
