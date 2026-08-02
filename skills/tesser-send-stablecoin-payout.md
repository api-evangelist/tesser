---
name: tesser-send-stablecoin-payout
description: Use when a developer wants to send a stablecoin payout to a counterparty via the Tesser API — register the counterparty and a funding wallet, create the payment, sign any wallet-originated steps, and poll to completion.
api: openapi/tesser-openapi-original.json
method: generated
operations: [counterparty_create, account_createWalletAccount, payments_createPayment, payments_signPaymentStep, payments_getPaymentById]
---

# Send a stablecoin payout on Tesser

Tesser moves money for licensed financial institutions on stablecoin rails. A
payout is created, may fan out into signed **steps**, and progresses through a
lifecycle you observe via polling or webhooks.

## Prerequisites

- A workspace Client ID + Client Secret (Dashboard -> Settings > API Keys).
- An access token: `POST https://auth.tesser.xyz/oauth/token` with
  `grant_type=client_credentials`, `audience=https://api.tesser.xyz`. Send the
  returned JWT as `Authorization: Bearer <token>` on every call.
- Base URL: `https://api.tesser.xyz` (production) or `https://sandbox.tesserx.co`
  (sandbox).

## Steps

1. **Register the payee** — `counterparty_create` (`POST /v1/entities/counterparties`).
   Pre-register the external individual/business you are paying.
2. **Register a funding wallet** — `account_createWalletAccount`
   (`POST /v1/accounts/wallets`) if you are paying from a self-custodial wallet.
   For wallet accounts, use the SDK (`@tesser-payments/sdk-ts` or `xyz.tesser:sdk`)
   to locally stamp the `signCreateWallet` activity.
3. **Create the payment** — `payments_createPayment` (`POST /v1/payments`) with the
   `direction`, funding account, and the `desired` from/to (account, amount,
   currency, network). Read back the `desired`/`estimated` overlays.
4. **Sign wallet-originated steps** — for each returned step that needs a
   signature, call `payments_signPaymentStep`
   (`POST /v1/payments/{paymentId}/steps/{stepId}/sign`) with the signature your
   local signer produced. Custodian-originated steps do not need this.
5. **Poll to completion** — `payments_getPaymentById`
   (`GET /v1/payments/{paymentId}`) until the payment reaches a terminal status,
   or subscribe to `payment.*` / `step.*` webhooks instead of polling.

## Rules

- **Auth:** JWT bearer, 24h lifetime; refresh via the token endpoint.
- **Pagination:** list endpoints accept `page` and `limit`.
- **Errors:** non-2xx returns `{ "errors": [ { "error_code": "domain-YZZZ",
  "error_message": "..." } ] }`. Failed steps carry `status_reasons[]` reusing the
  same vocabulary. See errors/tesser-error-codes.yml.
- **Never invent shapes** — fetch the live OpenAPI at
  `https://docs.tesser.xyz/api/v1/schema.json` before constructing a write.
- **Secrets:** never print client secrets or wallet private keys.
