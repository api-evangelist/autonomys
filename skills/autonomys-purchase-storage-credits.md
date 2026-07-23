---
name: Purchase Auto Drive storage credits with AI3
description: Programmatically buy Autonomys Auto Drive storage credits by creating a purchase intent, paying on-chain with AI3, and confirming settlement.
api: openapi/autonomys-auto-drive-openapi.json
base_url: https://mainnet.auto-drive.autonomys.xyz
operations:
  - GET /intents/contract
  - GET /intents/price
  - POST /intents
  - POST /intents/{id}/watch
  - GET /intents/{id}
  - GET /subscriptions/credits
---

# Purchase Auto Drive storage credits with AI3

Third-party apps top up storage credits by locking a price in an intent, paying the smart contract in native AI3, then submitting the transaction hash for settlement. Requires an API key from https://ai3.storage.

## Auth
- `Authorization: Bearer <api-key>`
- `X-Auth-Provider: apikey`

## Steps
1. **Get contract info** — `GET /intents/contract` returns the contract address, chain ID, and ABI.
2. **Check price** — `GET /intents/price` returns the current price per byte and per GB to show the user.
3. **Create the intent** — `POST /intents` returns an `intentId` with the price locked in.
4. **Pay on-chain** — call `payIntent(intentId)` on the contract, sending AI3 as native value.
5. **Submit the tx hash** — `POST /intents/{id}/watch` with the transaction hash.
6. **Confirm** — poll `GET /intents/{id}` until settled, then verify the new balance with `GET /subscriptions/credits`.

## Rules
- The `intentId` is the idempotency anchor for the whole flow — reuse it across watch/poll; do not create a new intent to retry payment.
- The locked price expires; if the intent lapses, create a fresh one via `POST /intents`.
- Errors are plain HTTP status codes with a description (see errors/autonomys-problem-types.yml).
