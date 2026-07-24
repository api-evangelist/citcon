---
name: Charge and capture a payment with Citcon UPI
description: >-
  Authenticate to the Citcon Universal Payment Interface, create a charge for a
  supported payment method, confirm/authorize it, and capture the funds — with
  correct idempotency references and error handling.
api: Citcon Universal Payment Interface (UPI) API
source: https://www.citcon.com/dev/universal-payment-interface-upi---api
generated: '2026-07-18'
method: generated
operations:
- POST /v1/access-tokens
- POST /v1/charges
- POST /v1/charges/{charge-token}
- POST /v1/captures
- GET /v1/transactions/{id}
---

# Charge and capture a payment with Citcon UPI

Base URLs: sandbox `https://api.sandbox.citconpay.com`, production
`https://api.citconpay.com/v1/`. All requests are JSON (`Content-Type: application/json`).

## 1. Get an access token

`POST /v1/access-tokens` using your Citcon private key as a Bearer credential:

```
Authorization: Bearer {private-key}
```

The response returns an access token (default expiry 24 hours) with a
`permissions` array such as `["charge","inquiry","capture","vault","consult","encryption-config","refund"]`.
Use this token as `Authorization: Bearer {access-token}` on every subsequent call.
A `401 Unauthorized` (or error code `4010`) means the token is missing/expired —
re-issue it.

## 2. Create the charge

`POST /v1/charges` with a **unique `reference`** (idempotency key — Citcon rejects
duplicates with error `4100 "duplicate request"`), plus `amount` (smallest currency
unit, e.g. cents), `currency` (ISO 4217), `country` (ISO 3166), and
`payment.method` (`card`, `alipay`, `wechatpay`, `klarna`, …) with
`payment.indicator` (`authenticated`, `non_authenticated`, `installment`,
`recurring`, `bnpl`). Include callback URLs under `urls` (`ipn`, `success`,
`fail`, `cancel`). The response returns a `charge-token`.

## 3. Confirm / authorize

`POST /v1/charges/{charge-token}` to confirm the authorization for methods that
require a confirmation step.

## 4. Capture

`POST /v1/captures` to capture the authorized funds. The **capture reference must
differ from the original charge reference** and be unique within the merchant
account.

## 5. Verify

`GET /v1/transactions/{id}` to inquire the final status, or rely on the `ipn`
webhook (server-to-server POST) for the asynchronous status update.

## Error handling

Errors return `status: "fail"` with `data: { code, message }`. Handle at least:
`4100` duplicate request (reuse a new reference), `4010` unauthorized (re-auth),
`4208` transaction ineligible for partial refund. See
`errors/citcon-problem-types.yml` and `conventions/citcon-conventions.yml`.

Test with sandbox values in `sandbox/citcon-sandbox.yml` (e.g. Visa
`4111111111111111`).
