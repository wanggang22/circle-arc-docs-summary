# Circle Mint API Reference - Complete Summary

> Generated from 63 Circle Developer Documentation pages
> Base URLs: `https://api-sandbox.circle.com` (Sandbox) | `https://api.circle.com` (Production)
> Authentication: Bearer token required on all endpoints unless noted otherwise

---

## Table of Contents

1. [General / Management](#1-general--management)
2. [Account - Balances](#2-account---balances)
3. [Account - Deposit Addresses](#3-account---deposit-addresses)
4. [Account - Deposits](#4-account---deposits)
5. [Account - Recipient Addresses](#5-account---recipient-addresses)
6. [Account - Transfers (Blockchain)](#6-account---transfers-blockchain)
7. [Account - Payouts (Fiat Off-ramp)](#7-account---payouts-fiat-off-ramp)
8. [Account - Wire Bank Accounts](#8-account---wire-bank-accounts)
9. [Account - CUBIX Bank Accounts](#9-account---cubix-bank-accounts)
10. [Account - PIX Bank Accounts](#10-account---pix-bank-accounts)
11. [Account - Burn Fee Calculations](#11-account---burn-fee-calculations)
12. [Account - Mock Endpoints (Sandbox)](#12-account---mock-endpoints-sandbox)
13. [Cross-Currency / FX](#13-cross-currency--fx)
14. [Payments (Crypto Payment Intents)](#14-payments-crypto-payment-intents)
15. [Payouts (Address Book)](#15-payouts-address-book)
16. [Institutional (External Entities)](#16-institutional-external-entities)
17. [Reserve Management](#17-reserve-management)
18. [Notifications (Subscriptions)](#18-notifications-subscriptions)
19. [Common Patterns](#19-common-patterns)

---

## 1. General / Management

### 1.1 Ping (Health Check)

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/ping` |
| Auth | None |
| Response | `{ "message": "pong" }` |
| Status | `200` Success |

**Notes:** No authentication required. Simple health check endpoint.

---

### 1.2 Get Account Configuration

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/configuration` |
| Auth | Bearer token |
| Response | `{ data: { payments: { masterWalletId: string } } }` |
| Status | `200` Success |

**Notes:** Returns the `masterWalletId` -- the system-generated identifier for the merchant's main wallet.

---

### 1.3 List All Stablecoins

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/stablecoins` |
| Auth | **None required** |
| Status | `200` Success, `429` Rate limited |

**Response Schema:**
```json
{
  "data": [
    {
      "name": "string",
      "symbol": "string",
      "totalAmount": "string",
      "chains": [
        { "amount": "string", "chain": "ETH|SOL|ALGO|..." }
      ]
    }
  ]
}
```

**Notes:**
- Rate limited to **1 call per minute** (IP-based)
- Returns per-chain circulating supply breakdown
- 30+ supported chains: ALGO, APTOS, ARB, AVAX, BASE, BTC, CELO, CODEX, ETH, HBAR, HYPEREVM, INK, LINEA, NEAR, NOBLE, OP, PLUME, PAH, POLY, SEI, SOL, SONIC, SUI, UNI, WORLDCHAIN, XDC, XLM, XRP, ZKS, ZKSYNC

---

## 2. Account - Balances

### 2.1 List All Balances

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/balances` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletId` | string | No | Defaults to main wallet if omitted |

**Response Schema:**
```json
{
  "data": {
    "available": [{ "amount": "3.14", "currency": "USD" }],
    "unsettled": [{ "amount": "3.14", "currency": "USD" }]
  }
}
```

**Currencies:** USD, EUR, BTC, ETH

---

## 3. Account - Deposit Addresses

### 3.1 Create a Deposit Address

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/wallets/addresses/deposit` |
| Auth | Bearer token |
| Status | `201` Created, `400`, `401` |

**Query Params:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletId` | string | No | Wallet to credit; defaults to main wallet |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `currency` | enum | Yes | USD, EUR, BTC, ETH |
| `chain` | enum | Yes | 30+ chains supported |

**Response:**
```json
{
  "data": {
    "address": "0x...",
    "addressTag": "string or null",
    "currency": "USD",
    "chain": "ETH"
  }
}
```

**Notes:**
- Circle may reuse addresses on blockchains that support reuse
- XRP: only classic address format (not x-address)
- Circle Mint Singapore customers must verify all transfer recipients via UI

### 3.2 List All Deposit Addresses

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/wallets/addresses/deposit` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Params:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletId` | string | No | Defaults to main wallet |

**Response:** Array of `{ address, addressTag, currency, chain }` objects.

---

## 4. Account - Deposits

### 4.1 Get a Deposit by ID

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/deposits/{id}` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

**Query Params:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | No | Filter: `wire` |
| `walletId` | string | No | Defaults to main wallet |

**Response - BusinessDeposit:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Deposit identifier |
| `sourceWalletId` | UUID | Source bank account |
| `destination` | object | Wallet location (type, id) |
| `amount` | FiatMoney | amount + currency (USD/EUR/MXN/SGD/BRL) |
| `fee` | FiatMoneyUsd | Fee in USD |
| `status` | enum | pending, complete, failed |
| `riskEvaluation` | object | approved/denied/review + reason |
| `customerExternalRef` | string | External reference from memo |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

### 4.2 List All Deposits

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/deposits` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | enum | No | `wire` |
| `walletId` | UUID | No | Defaults to main wallet |
| `from` | date-time | No | Inclusive start |
| `to` | date-time | No | Inclusive end |
| `pageAfter` | string | No | Pagination cursor |
| `pageBefore` | string | No | Pagination cursor |
| `pageSize` | integer | No | Items per page |

**Notes:**
- Returns max 50 deposits in descending chronological order
- Omitting date params returns most recent deposits

---

## 5. Account - Recipient Addresses

### 5.1 Create a Recipient Address

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/wallets/addresses/recipient` |
| Auth | Bearer token |
| Status | `200` Success, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `address` | string | Yes | Blockchain address (preserve exact formatting) |
| `chain` | enum | Yes | 30+ chains |
| `description` | string | Yes | Human-readable identifier |
| `addressTag` | string | No | Secondary identifier (e.g., Stellar memo) |
| `currency` | enum | No | USD, EUR, BTC, ETH (defaults to USD; BTC for Bitcoin) |

**Response:**
```json
{
  "data": {
    "id": "UUID",
    "address": "0x...",
    "addressTag": "...",
    "chain": "ETH",
    "currency": "USD",
    "description": "...",
    "status": "active|pending_verification|verification_succeeded"
  }
}
```

**Notes:**
- Newly added recipient addresses **must be verified** before use
- Circle Mint France customers must verify all transfer recipients via Circle Console UI
- XRP: only classic address format

### 5.2 List All Recipient Addresses

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/wallets/addresses/recipient` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:** `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` (standard pagination)

**Notes:** Addresses pending administrator verification are **not included** in the response.

### 5.3 Delete a Recipient Address

| Detail | Value |
|--------|-------|
| Method | `DELETE` |
| Path | `/v1/businessAccount/wallets/addresses/recipient/{id}` |
| Auth | Bearer token |
| Status | `200`, `400`, `401`, `404` |

**Notes:** Recipient must be in "active" or "pending" state to be deleted.

---

## 6. Account - Transfers (Blockchain)

### 6.1 Create a Transfer

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/transfers` |
| Auth | Bearer token |
| Status | `201` Created, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `destination` | object | Yes | `type: "verified_blockchain"`, `addressId: UUID` |
| `amount` | Money | Yes | amount (string) + currency (USD/EUR/BTC/ETH) |
| `source` | object | No | `type: "wallet"`, `id: string`. Defaults to main wallet |

**Response - Transfer Object:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Transfer identifier |
| `source` | object | Wallet or blockchain source |
| `destination` | object | Blockchain or wallet destination |
| `amount` | Money | Transferred amount |
| `fees` | array | Fee objects (amount, currency, type) |
| `transactionHash` | string | Onchain tx hash |
| `status` | enum | pending, complete, failed |
| `errorCode` | enum | transfer_failed, transfer_denied, blockchain_error, insufficient_funds |
| `createDate` | ISO-8601 | |

**Important Rules:**
- Identity required when: destination is blockchain AND chain is ETH AND amount >= $3,000
- XRP: classic address format only
- 40+ blockchains supported

### 6.2 Get a Transfer

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/transfers/{id}` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

Same response schema as Create Transfer.

### 6.3 List All Transfers

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/transfers` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletId` | string | No | Filter to/from wallet |
| `sourceWalletId` | string | No | Filter from wallet |
| `destinationWalletId` | string | No | Filter to wallet |
| `from` | date-time | No | Inclusive start |
| `to` | date-time | No | Inclusive end |
| `pageAfter` | string | No | Pagination |
| `pageBefore` | string | No | Pagination |
| `pageSize` | integer | No | Min: 1 |

**Notes:** Returns up to 50 transfers in descending chronological order.

---

## 7. Account - Payouts (Fiat Off-ramp)

### 7.1 Create a Payout (Business Account)

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/payouts` |
| Auth | Bearer token |
| Status | `201` Created, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once |
| `destination` | object | Yes | `type` (wire/cubix/pix/sepa/sepa_instant) + `id` |
| `amount` | FiatMoney | Yes | amount (string) + currency (USD/EUR/MXN/SGD/BRL) |
| `toAmount` | object | No | Currency exchange spec |
| `source` | object | No | `type: "wallet"`, `id`. Defaults to main wallet |

**Response - BusinessPayout:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `sourceWalletId` | string | |
| `destination` | object | type, id, name |
| `amount` | FiatMoney | |
| `toAmount` | FiatMoney | Converted amount |
| `fees` | object | |
| `status` | enum | pending, complete, failed |
| `trackingRef` | string | |
| `errorCode` | enum | insufficient_funds, transaction_denied, transaction_failed, transaction_returned, bank_transaction_error, fiat_account_limit_exceeded, invalid_bank_account_number, invalid_ach_rtn, invalid_wire_rtn, vendor_inactive |
| `riskEvaluation` | object | approval/denial + reason |
| `adjustments` | object | FX credits/debits |
| `return` | object | Present only if payout returned by bank |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

**Notes:** Converts digital assets to fiat (redemption/offramp). All monetary amounts use string format.

### 7.2 Get a Payout (Business Account)

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/payouts/{id}` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

Same response schema as Create Payout (BusinessPayout).

### 7.3 List All Payouts (Business Account)

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/payouts` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `destination` | UUID | No | Filter by bank account ID |
| `type` | string | No | `wire` or `cubix` |
| `status` | array | No | pending, complete, failed |
| `sourceWalletId` | UUID | No | Filter by source wallet |
| `from` | date-time | No | Inclusive start |
| `to` | date-time | No | Inclusive end |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |
| `pageSize` | integer | No | Min: 1 |

**Important:** This endpoint does **NOT** return tracking reference numbers. Use the individual GET endpoint for that.

---

## 8. Account - Wire Bank Accounts

### 8.1 Create a Wire Bank Account

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/banks/wires` |
| Auth | Bearer token |
| Status | `201` Created, `400`, `401` |

**Three account types (discriminated union):**

**Type A - US Bank Account:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `accountNumber` | string (6-35 chars) | Yes |
| `routingNumber` | string (ABA) | Yes |
| `billingDetails` | object | Yes |
| `bankAddress` | object | Yes |

**Type B - IBAN Account:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `iban` | string | Yes |
| `billingDetails` | object | Yes |
| `bankAddress` | object | Yes (city + country min) |

**Type C - Non-IBAN International:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `accountNumber` | string (6-35 chars) | Yes |
| `routingNumber` | string (SWIFT/BIC) | Yes |
| `billingDetails` | object | Yes |
| `bankAddress` | object | Yes (bankName + city + country) |

**BillingDetails (shared):**
| Field | Type | Required |
|-------|------|----------|
| `name` | string (max 1024) | Yes |
| `city` | string (max 1024) | Yes |
| `country` | string (2-char ISO) | Yes |
| `line1` | string (max 1024) | Yes |
| `postalCode` | string (max 16) | Yes |
| `line2` | string | No |
| `district` | string | Required for US/Canada |

**Response - Wire Object:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `status` | enum | pending, complete, failed |
| `description` | string | Bank name + masked account (e.g., "WELLS FARGO BANK, NA ****0010") |
| `trackingRef` | string | Reference for wire beneficiary field |
| `transferTypesInfo` | object | Supported transfer types + currencies |
| `fingerprint` | UUID | Uniquely identifies account number |
| `billingDetails` | object | |
| `bankAddress` | object | |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

**Supported Currencies:** USD, EUR, MXN, SGD, BRL

### 8.2 Get a Wire Bank Account

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/wires/{id}` |
| Status | `200`, `401`, `404` |

### 8.3 List All Wire Bank Accounts

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/wires` |
| Status | `200`, `401` |

No query parameters. Returns array of Wire objects.

### 8.4 Get Wire Instructions

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/wires/{id}/instructions` |
| Status | `200`, `401`, `404` |

**Query Params:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `currency` | enum | No | USD or EUR (defaults to USD) |
| `walletId` | string | No | External entity wallet |

**Response - WireInstruction:**
```json
{
  "data": {
    "trackingRef": "string",
    "beneficiary": { "name": "...", "address1": "...", "address2": "..." },
    "beneficiaryBank": {
      "name": "...", "swiftCode": "...", "routingNumber": "...",
      "accountNumber": "...(masked)", "currency": "...",
      "address": "...", "city": "...", "postalCode": "...", "country": "..."
    }
  }
}
```

---

## 9. Account - CUBIX Bank Accounts

### 9.1 Create a CUBIX Bank Account

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/banks/cubix` |
| Auth | Bearer token |
| Status | `201` Created, `400`, `401` |

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `accountId` | UUID | Yes (CUBIX Account ID) |

**Response - CubixFiatAccountResponse:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `status` | enum | pending, complete, failed |
| `trackingRef` | string | For public description field |
| `accountId` | UUID | CUBIX Account ID |
| `transferTypesInfo` | object | Currencies: USD, EUR, MXN, SGD, BRL |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

### 9.2 Get a CUBIX Bank Account

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/cubix/{id}` |
| Status | `200`, `401`, `404` |

### 9.3 List All CUBIX Bank Accounts

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/cubix` |
| Status | `200`, `401` |

No parameters. Returns array of CUBIX account objects.

### 9.4 Get CUBIX Instructions

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/cubix/{id}/instructions` |
| Status | `200`, `401`, `404` |

**Response:**
```json
{
  "data": {
    "trackingRef": "CIR25XSXT8",
    "accountId": "UUID"
  }
}
```

**Notes:** `trackingRef` must be set in the CUBIX public memo field. `accountId` must be set in the CUBIX account ID field.

---

## 10. Account - PIX Bank Accounts

### 10.1 Create a PIX Bank Account

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/businessAccount/banks/pix` |
| Auth | Bearer token |
| Status | `201` Created, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | |
| `name` | string | Yes | Beneficiary name |
| `accountNumber` | string | Yes | Beneficiary account number |
| `ispb` | string | Yes | ISPB code |
| `branchCode` | string | Yes | Branch code |
| `taxId` | string | Yes | Tax ID |
| `accountType` | enum | Yes | checking, savings, salary, prepaid |

**Response - PixFiatAccountResponse:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `status` | enum | pending, complete, failed |
| `description` | string | Bank name + last 4 digits |
| `trackingRef` | string | |
| `transferTypesInfo` | object | PIX only, BRL currency |
| `riskEvaluation` | object | approved/denied/review |
| `fingerprint` | UUID | |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

### 10.2 Get a PIX Bank Account

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/pix/{id}` |
| Status | `200`, `401`, `404` |

### 10.3 List All PIX Bank Accounts

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/pix` |
| Status | `200`, `401` |

### 10.4 Get PIX Instructions

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/businessAccount/banks/pix/{id}/instructions` |
| Status | `200`, `401`, `404` |

**Response - PixInstruction:**
```json
{
  "data": {
    "trackingRef": "string",
    "ispb": "string",
    "branchCode": "string",
    "accountNumber": "string",
    "accountType": "checking|savings|salary|prepaid",
    "taxId": "string",
    "name": "string"
  }
}
```

---

## 11. Account - Burn Fee Calculations

### 11.1 List Daily Burn Fee Calculations

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/fees/redemption/dailyReports` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `feeType` | enum | **Yes** | `gross` or `net` |
| `minimumFeeAmount` | string | No | Filter out below threshold |
| `currency` | enum | No | USD or EUR (default: USD) |
| `from` | date-time | No | Inclusive start |
| `to` | date-time | No | Inclusive end |
| `pageSize` | integer | No | Min: 1, max: 50 default |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |

**Response - RedemptionFeeCalculation:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `fee` | object | amount + currency |
| `cumulatedPayoutAmount` | object | |
| `cumulatedPaymentAmount` | object | |
| `cumulatedNetAmount` | object | |
| `valueDate` | string | |
| `status` | enum | scheduled, pending, paid |
| `thresholdResetTimestamp` | string | |
| `createDate` | ISO-8601 | |
| `feeType` | string | |

**Notes:** Returns up to 50 calculations in descending chronological order. Supports USD, EUR, MXN, SGD, BRL.

---

## 12. Account - Mock Endpoints (Sandbox)

### 12.1 Create a Mock Wire Payment

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/mocks/payments/wire` |
| Auth | Bearer token |
| Status | `201`, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `trackingRef` | string | Yes | Wire reference (set in beneficiary field) |
| `amount` | object | Yes | `{ amount: "3.14", currency: "USD" }` |
| `beneficiaryBank` | object | Yes | `{ accountNumber: "string" }` |

**Response:**
```json
{
  "data": {
    "trackingRef": "string",
    "amount": { "amount": "string", "currency": "USD" },
    "beneficiaryBank": { "accountNumber": "string" },
    "status": "pending|processed|failed"
  }
}
```

### 12.2 Create a Mock PIX Payment

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/mocks/payments/pix` |
| Auth | Bearer token |
| Status | `201`, `400`, `401` |

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `trackingRef` | string | Yes |
| `amount` | object | Yes (`currency: "BRL"` only) |
| `accountNumber` | string | Yes |

**Response:**
```json
{
  "data": {
    "trackingRef": "string",
    "amount": { "amount": "string", "currency": "BRL" },
    "beneficiaryAccountNumber": "string",
    "status": "pending|processed|failed"
  }
}
```

---

## 13. Cross-Currency / FX

### 13.1 Create FX Account

| Detail | Value |
|--------|-------|
| Method | `PUT` |
| Path | `/v1/exchange/fxConfigs/accounts` |
| Auth | Bearer token |
| Status | `200`, `400`, `401`, `404` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fiatAccountId` | UUID | Yes | Fiat currency account identifier |
| `currency` | enum | Yes | MXN or BRL |

**Response:**
```json
{
  "data": {
    "currency": "MXN|BRL",
    "fiatAccountId": "UUID",
    "createDate": "ISO-8601",
    "updateDate": "ISO-8601"
  }
}
```

### 13.2 Get Daily FX Limits

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/exchange/fxConfigs/dailyLimits` |
| Auth | Bearer token |
| Status | `200`, `401` |

No parameters required.

**Response:**
```json
{
  "data": {
    "EURC": { "limit": "1000000.00", "usage": "0.00", "available": "1000000.00" },
    "MXN": { "limit": "...", "usage": "...", "available": "..." },
    "USDC": { "limit": "...", "usage": "...", "available": "..." },
    "BRL": { "limit": "...", "usage": "...", "available": "..." }
  }
}
```

### 13.3 Get a Quote

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/exchange/quotes` |
| Auth | Bearer token |
| Status | `201`, `400`, `401`, `404` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | |
| `type` | enum | Yes | `reference` or `tradable` |
| `from` | object | Yes | Source currency + optional amount |
| `to` | object | Yes | Target currency + optional amount |

**Supported Currencies:** USD, EUR, MXN, SGD, BRL, USDC, EURC, HKD, GBP, CNH, AED

**Response:**
```json
{
  "data": {
    "id": "UUID",
    "rate": 20.15,
    "from": { "amount": "100.00", "currency": "USD" },
    "to": { "amount": "2015.00", "currency": "MXN" },
    "expiry": "ISO-8601",
    "type": "tradable"
  }
}
```

**Important:** Either the `from` or `to` currency must be USD.

### 13.4 Create FX Trade

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/exchange/trades` |
| Auth | Bearer token |
| Status | `200`, `400`, `401`, `404` |

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `quoteId` | UUID | Yes |

**Response - Trade Object:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `from` | FxMoney | amount + currency (USDC/EURC/MXN/BRL) |
| `to` | FxMoney | amount + currency |
| `status` | enum | pending, confirmed, failed, complete, pending_settlement |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |
| `quoteId` | UUID | |

### 13.5 Get an FX Trade

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/exchange/trades/{id}` |
| Status | `200`, `401`, `404` |

Same response schema as Create FX Trade, plus `settlementId`.

### 13.6 Get All FX Trades

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/exchange/trades` |
| Status | `200`, `401`, `404` |

**Query Params:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `settlementId` | UUID | No | Filter by settlement |

### 13.7 Get a Settlement

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/exchange/trades/settlements/{id}` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

**Response - Settlement:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `entityId` | UUID | |
| `status` | enum | pending, settled |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |
| `details` | array | Line items: id, type (payable/receivable), amount, status (pending/completed), reference, timestamps |

### 13.8 Get All Settlements

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/exchange/trades/settlements` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `from` | date-time | No | |
| `to` | date-time | No | |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |
| `pageSize` | integer | No | Min: 1 |
| `type` | enum | No | account_payable, account_receivable |
| `status` | enum | No | pending, settled |
| `currency` | enum | No | MXN, BRL |

### 13.9 Get Settlement Instructions

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/exchange/trades/settlements/instructions/{currency}` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

**Path Param:** `currency` - `BRL` or `MXN`

**MXN Response:** Wire details (trackingRef, beneficiary, bank info)
**BRL Response:** PIX details (ISPB, branchCode, accountNumber, accountType, taxId)

---

## 14. Payments (Crypto Payment Intents)

### 14.1 Create a Payment Intent

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/paymentIntents` |
| Auth | Bearer token |
| Status | `201`, `400`, `401`, `403`, `404` |

**Two types supported (oneOf):**

**Transient Payment Intent:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `amount` | Money | Yes |
| `settlementCurrency` | enum | Yes (USD, EUR, BTC, ETH) |
| `paymentMethods` | array | Yes (`type: "blockchain"`, `chain`) |
| `merchantWalletId` | string | No |

**Continuous Payment Intent:**
| Field | Type | Required |
|-------|------|----------|
| `idempotencyKey` | UUID v4 | Yes |
| `currency` | enum | Yes (USD, EUR, BTC, ETH) |
| `settlementCurrency` | enum | Yes |
| `paymentMethods` | array | Yes |
| `merchantWalletId` | string | Yes |
| `type` | string | Yes (`"continuous"`) |

**Response:** PaymentIntent or ContinuousPaymentIntent with: id, amounts, settlement currency, payment methods, timeline, fees, timestamps.

### 14.2 Get a Payment Intent

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/paymentIntents/{id}` |
| Status | `200`, `401`, `404` |

Returns transient or continuous payment intent with full details including: amount, amountPaid, amountRefunded, settlementCurrency, paymentMethods, fees, paymentIds, refundIds, timeline, expiresOn.

### 14.3 List All Payment Intents

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/paymentIntents` |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | enum | No | created, pending, complete, expired, failed |
| `context` | enum | No | underpaid, paid, overpaid |
| `from` | date-time | No | |
| `to` | date-time | No | |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |
| `pageSize` | integer | No | Min: 1 |

**Notes:** Filters by most recent `timeline.status` within the payment intent.

### 14.4 Expire a Payment Intent

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/paymentIntents/{id}/expire` |
| Auth | Bearer token |
| Status | `201`, `400`, `401`, `404` |

**Request Body:** Empty JSON object `{}`

Returns the expired payment intent with updated timeline status.

### 14.5 Refund a Payment Intent

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/paymentIntents/{id}/refund` |
| Auth | Bearer token |
| Status | `201`, `400`, `401`, `404` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | |
| `destination` | object | Yes | address, chain, optional addressTag |
| `amount` | Money | Yes | Source amount + currency |
| `toAmount` | Money | Yes | Destination amount + currency |

**Response:** CryptoPayment object with id, type (refund), status, amounts, depositAddress, transactionHash, etc.

### 14.6 Get a Payment

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/payments/{id}` |
| Auth | Bearer token |
| Status | `200`, `401`, `404` |

Returns polymorphic payment: either FiatPaymentPolymorphic or CryptoPayment.

**Payment statuses:** pending, confirmed, paid, failed, action_required
**Terminal states:** `paid` and `failed`

### 14.7 List All Payments

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/payments` |
| Auth | Bearer token |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | UUID | No | Filter by source ID |
| `settlementId` | UUID | No | |
| `paymentIntentId` | UUID | No | |
| `type` | string[] | No | card |
| `status` | enum | No | pending, confirmed, paid, failed, action_required |
| `from` | date-time | No | |
| `to` | date-time | No | |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |
| `pageSize` | integer | No | Min: 1 |

**Return types:** FiatPayment, CryptoPayment, FiatCancel, FiatRefund

---

## 15. Payouts (Address Book)

### 15.1 Create a Recipient (Address Book)

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/addressBook/recipients` |
| Auth | Bearer token |
| Status | `201`, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | |
| `chain` | enum | Yes | 19+ chains: ALGO, APTOS, ARB, AVAX, BASE, CELO, ETH, HBAR, NEAR, NOBLE, OP, PAH, POLY, SOL, SUI, UNI, XLM, XRP, ZKS |
| `address` | string | Yes | Preserve exact formatting/capitalization |
| `addressTag` | string | No | Secondary identifier (e.g., Stellar memo) |
| `metadata` | object | Yes | `nickname`, `email` (max 1024), `bns` (Blockchain Name Service) |

**Response - AddressBookRecipient:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `chain` | string | |
| `address` | string | |
| `addressTag` | string | |
| `metadata` | object | nickname, email, bns |
| `status` | enum | pending, inactive, active, denied |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

### 15.2 Get a Recipient

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/addressBook/recipients/{id}` |
| Status | `200`, `401`, `404` |

### 15.3 List All Recipients

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/addressBook/recipients` |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `address` | string | No | Filter by blockchain address |
| `chain` | string | No | Filter by chain |
| `email` | string | No | Filter by metadata email |
| `status` | enum | No | pending, inactive, active, denied |
| `from` | date-time | No | |
| `to` | date-time | No | |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |
| `pageSize` | integer | No | |

### 15.4 Modify a Recipient

| Detail | Value |
|--------|-------|
| Method | `PATCH` |
| Path | `/v1/addressBook/recipients/{id}` |
| Auth | Bearer token |
| Status | `200`, `400`, `401`, `404` |

**Request Body:**
```json
{
  "metadata": {
    "nickname": "string",
    "email": "string (max 1024)",
    "bns": "string"
  }
}
```

**Notes:** Only metadata fields are modifiable.

### 15.5 Delete a Recipient

| Detail | Value |
|--------|-------|
| Method | `DELETE` |
| Path | `/v1/addressBook/recipients/{id}` |
| Status | `200`, `400`, `401`, `404` |

### 15.6 Create a Payout (Address Book)

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/payouts` |
| Auth | Bearer token |
| Status | `201`, `400`, `401` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | |
| `destination` | object | Yes | `type: "address_book"`, `id` (UUID) |
| `amount` | Money | Yes | currency: USD or EUR |
| `source` | object | No | Wallet (type + id) |
| `toAmount` | object | No | Target crypto amount |

**Response - CryptoPayout:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | |
| `sourceWalletId` | string | |
| `destination` | object | type + id |
| `amount` | Money | |
| `toAmount` | Money | Converted amount |
| `fees` | object | |
| `networkFees` | object | Blockchain fees |
| `status` | enum | pending, complete, failed |
| `errorCode` | enum | insufficient_funds, transaction_denied, transaction_failed, etc. |
| `riskEvaluation` | object | |
| `createDate` | ISO-8601 | |
| `updateDate` | ISO-8601 | |

**Important:** Identities required when destination is blockchain, chain is ETH, or amount >= $3,000.

### 15.7 Get a Payout (Address Book)

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/payouts/{id}` |
| Status | `200`, `401`, `404` |

Currencies: USD, EUR, BTC, ETH, MTC, FLW, MAN

### 15.8 List All Payouts (Address Book)

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/payouts` |
| Status | `200`, `401` |

**Query Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | string | No | Source wallet filter |
| `destination` | UUID | No | Destination filter |
| `type` | array | No | Destination type (multi-value) |
| `status` | array | No | pending, complete, failed |
| `sourceCurrency` | enum | No | USD, EUR, BTC, ETH, MTC, FLW, MAN |
| `destinationCurrency` | enum | No | Same options |
| `chain` | string | No | 30+ chains |
| `from` | date-time | No | |
| `to` | date-time | No | |
| `pageAfter` | string | No | |
| `pageBefore` | string | No | |
| `pageSize` | integer | No | Min: 1 |

---

## 16. Institutional (External Entities)

### 16.1 Create an External Entity

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/externalEntities` |
| Auth | Bearer token |
| Status | `201`, `400`, `401`, `409` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `businessName` | string | Yes | |
| `businessUniqueIdentifier` | string | Yes | Company number or tax ID |
| `identifierIssuingCountryCode` | string | Yes | ISO 3166-1 alpha-2 |
| `address` | object | Yes | country, state, city, postalCode (required); streetName, buildingNumber (optional) |

**Response:**
```json
{
  "data": {
    "walletId": "string",
    "businessName": "string",
    "businessUniqueIdentifier": "string",
    "identifierIssuingCountryCode": "string",
    "complianceState": "PENDING|ACCEPTED|REJECTED"
  }
}
```

**Notes:**
- Requires institutional account access -- contact Circle account representative
- System generates `walletId` for the entity
- 409 Conflict if entity already exists

### 16.2 Get All External Entities

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/externalEntities` |
| Status | `200`, `401` |

**Query Params:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `businessUniqueIdentifier` | string | No | Company number/tax ID |
| `identifierIssuingCountryCode` | string | No | ISO alpha-2 |

**Important:** Both `businessUniqueIdentifier` and `identifierIssuingCountryCode` must be provided together, or not at all.

### 16.3 Get External Entity by Wallet ID

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/externalEntities/{walletId}` |
| Status | `200`, `401` |

Returns single ExternalEntity object.

---

## 17. Reserve Management

### 17.1 Report Daily Custody Balances

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v2/reserveManagement/dailyCustodyBalances` |
| Auth | Bearer token |
| Status | `201` (new), `200` (duplicate idempotency key), `400`, `401`, `403`, `409` |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | |
| `currency` | enum | Yes | USDC or EURC |
| `localBalance` | number | Yes | Regional token balance |
| `asOfDate` | date | Yes | YYYY-MM-DD UTC |
| `reportType` | enum | Yes | `eea` or `japan_trust` |
| `additionalFields` | object | Conditional | Required for `eea` reports only |

**EEA Additional Fields:**
| Field | Type | Required |
|-------|------|----------|
| `equivalentEuroLocalBalance` | number | Yes |
| `totalBalance` | number | Yes |
| `equivalentEuroTotalBalance` | number | Yes |

**Response:**
```json
{
  "data": {
    "idempotencyKey": "UUID",
    "id": "UUID",
    "createDate": "ISO-8601",
    "localBalance": 1000000,
    "currency": "USDC",
    "asOfDate": "2024-01-15",
    "reportType": "eea",
    "additionalFields": { ... }
  }
}
```

**Important:** Only one report is allowed per day per currency.

---

## 18. Notifications (Subscriptions)

### 18.1 Create a Notification Subscription

| Detail | Value |
|--------|-------|
| Method | `POST` |
| Path | `/v1/notifications/subscriptions` |
| Auth | Bearer token |
| Status | `200`, `400`, `401`, `429` |

**Request Body:**
```json
{
  "endpoint": "https://example.org/handler/for/notifications"
}
```

**Response:**
```json
{
  "data": {
    "id": "UUID",
    "endpoint": "https://...",
    "subscriptionDetails": [
      {
        "url": "arn:aws:sns:...",
        "status": "confirmed|pending|deleted"
      }
    ]
  }
}
```

**Important Limits:**
- Sandbox: max **3** active subscriptions
- Production: max **1** active subscription
- 429 returned when limit exceeded
- Endpoint must handle **AWS SNS** subscription requests
- Must be publicly accessible HTTPS URL

### 18.2 List All Notification Subscriptions

| Detail | Value |
|--------|-------|
| Method | `GET` |
| Path | `/v1/notifications/subscriptions` |
| Status | `200`, `401` |

No parameters. Returns array of subscription objects.

### 18.3 Delete a Notification Subscription

| Detail | Value |
|--------|-------|
| Method | `DELETE` |
| Path | `/v1/notifications/subscriptions/{id}` |
| Status | `200`, `400`, `401`, `404` |

**Important:** Cannot delete a subscription with any `pending` status requests. All subscription request statuses must be either `confirmed`, `deleted`, or a combination.

---

## 19. Common Patterns

### Authentication
All endpoints (except `/ping` and `/v1/stablecoins`) require Bearer token authentication:
```
Authorization: Bearer <api_key>
```

### Idempotency
All mutating requests (POST/PUT/PATCH) require `idempotencyKey` (UUID v4) to ensure exactly-once execution.

### Pagination
Standard pagination parameters used across list endpoints:
| Param | Description |
|-------|-------------|
| `pageAfter` | Collection ID marking exclusive start |
| `pageBefore` | Collection ID marking exclusive end |
| `pageSize` | Items per page (min: 1, default varies) |

Use `pageAfter` OR `pageBefore`, not both simultaneously.

### Date Filtering
Most list endpoints support:
- `from` (date-time): Inclusive start, ISO-8601
- `to` (date-time): Inclusive end, ISO-8601

### Supported Currencies
| Context | Currencies |
|---------|-----------|
| Crypto | USD (USDC), EUR (EURC), BTC, ETH |
| Fiat | USD, EUR, MXN, SGD, BRL |
| FX | USDC, EURC, MXN, BRL, HKD, GBP, CNH, AED |

### Supported Blockchains (30+)
ALGO, APTOS, ARB, AVAX, BASE, BTC, CELO, CODEX, ETH, HBAR, HYPEREVM, INK, LINEA, NEAR, NOBLE, OP, PLUME, PAH, POLY, SEI, SOL, SONIC, SUI, UNI, WORLDCHAIN, XDC, XLM, XRP, ZKS, ZKSYNC

### Standard Status Values
| Resource | Statuses |
|----------|----------|
| Bank accounts | pending, complete, failed |
| Transfers | pending, complete, failed |
| Payouts | pending, complete, failed |
| Payments | pending, confirmed, paid, failed, action_required |
| Payment intents | created, pending, complete, expired, failed |
| FX Trades | pending, confirmed, failed, complete, pending_settlement |
| Settlements | pending, settled |
| Recipients | pending, inactive, active, denied |
| Subscriptions | confirmed, pending, deleted |

### Error Response Format
All error responses follow:
```json
{
  "code": 400,
  "message": "Human-readable error description"
}
```

### Response Headers
All responses include `X-Request-Id` (UUID v4) for tracking with Circle support.

### XRP Address Format
All endpoints: XRP uses **classic address format only** (e.g., `rPEPPER7kfTD9w2To4CQk6UCfuHM9c6GDY`). X-address format is NOT supported.

### Identity Requirements
Identity information required when:
- Destination type is blockchain AND chain is ETH AND amount >= $3,000

### Regional Requirements
- **Circle Mint France:** Must verify all transfer recipients via Circle Console UI, or transfers will be held pending
- **Circle Mint Singapore:** Must verify all transfer recipients using the UI; transfers may remain in `pending` status if recipients unverified

---

## Quick Reference: All Endpoints

| # | Method | Path | Feature |
|---|--------|------|---------|
| 1 | GET | `/ping` | Health check |
| 2 | GET | `/v1/configuration` | Account config |
| 3 | GET | `/v1/stablecoins` | Stablecoin supply |
| 4 | GET | `/v1/businessAccount/balances` | Balances |
| 5 | POST | `/v1/businessAccount/wallets/addresses/deposit` | Create deposit address |
| 6 | GET | `/v1/businessAccount/wallets/addresses/deposit` | List deposit addresses |
| 7 | GET | `/v1/businessAccount/deposits/{id}` | Get deposit |
| 8 | GET | `/v1/businessAccount/deposits` | List deposits |
| 9 | POST | `/v1/businessAccount/wallets/addresses/recipient` | Create recipient address |
| 10 | GET | `/v1/businessAccount/wallets/addresses/recipient` | List recipient addresses |
| 11 | DELETE | `/v1/businessAccount/wallets/addresses/recipient/{id}` | Delete recipient address |
| 12 | POST | `/v1/businessAccount/transfers` | Create transfer |
| 13 | GET | `/v1/businessAccount/transfers/{id}` | Get transfer |
| 14 | GET | `/v1/businessAccount/transfers` | List transfers |
| 15 | POST | `/v1/businessAccount/payouts` | Create payout (business) |
| 16 | GET | `/v1/businessAccount/payouts/{id}` | Get payout (business) |
| 17 | GET | `/v1/businessAccount/payouts` | List payouts (business) |
| 18 | POST | `/v1/businessAccount/banks/wires` | Create wire account |
| 19 | GET | `/v1/businessAccount/banks/wires/{id}` | Get wire account |
| 20 | GET | `/v1/businessAccount/banks/wires` | List wire accounts |
| 21 | GET | `/v1/businessAccount/banks/wires/{id}/instructions` | Wire instructions |
| 22 | POST | `/v1/businessAccount/banks/cubix` | Create CUBIX account |
| 23 | GET | `/v1/businessAccount/banks/cubix/{id}` | Get CUBIX account |
| 24 | GET | `/v1/businessAccount/banks/cubix` | List CUBIX accounts |
| 25 | GET | `/v1/businessAccount/banks/cubix/{id}/instructions` | CUBIX instructions |
| 26 | POST | `/v1/businessAccount/banks/pix` | Create PIX account |
| 27 | GET | `/v1/businessAccount/banks/pix/{id}` | Get PIX account |
| 28 | GET | `/v1/businessAccount/banks/pix` | List PIX accounts |
| 29 | GET | `/v1/businessAccount/banks/pix/{id}/instructions` | PIX instructions |
| 30 | GET | `/v1/fees/redemption/dailyReports` | Burn fee calculations |
| 31 | POST | `/v1/mocks/payments/wire` | Mock wire payment |
| 32 | POST | `/v1/mocks/payments/pix` | Mock PIX payment |
| 33 | PUT | `/v1/exchange/fxConfigs/accounts` | Create FX account |
| 34 | GET | `/v1/exchange/fxConfigs/dailyLimits` | Daily FX limits |
| 35 | POST | `/v1/exchange/quotes` | Get FX quote |
| 36 | POST | `/v1/exchange/trades` | Create FX trade |
| 37 | GET | `/v1/exchange/trades/{id}` | Get FX trade |
| 38 | GET | `/v1/exchange/trades` | List FX trades |
| 39 | GET | `/v1/exchange/trades/settlements/{id}` | Get settlement |
| 40 | GET | `/v1/exchange/trades/settlements` | List settlements |
| 41 | GET | `/v1/exchange/trades/settlements/instructions/{currency}` | Settlement instructions |
| 42 | POST | `/v1/paymentIntents` | Create payment intent |
| 43 | GET | `/v1/paymentIntents/{id}` | Get payment intent |
| 44 | GET | `/v1/paymentIntents` | List payment intents |
| 45 | POST | `/v1/paymentIntents/{id}/expire` | Expire payment intent |
| 46 | POST | `/v1/paymentIntents/{id}/refund` | Refund payment intent |
| 47 | GET | `/v1/payments/{id}` | Get payment |
| 48 | GET | `/v1/payments` | List payments |
| 49 | POST | `/v1/addressBook/recipients` | Create recipient |
| 50 | GET | `/v1/addressBook/recipients/{id}` | Get recipient |
| 51 | GET | `/v1/addressBook/recipients` | List recipients |
| 52 | PATCH | `/v1/addressBook/recipients/{id}` | Modify recipient |
| 53 | DELETE | `/v1/addressBook/recipients/{id}` | Delete recipient |
| 54 | POST | `/v1/payouts` | Create payout (address book) |
| 55 | GET | `/v1/payouts/{id}` | Get payout (address book) |
| 56 | GET | `/v1/payouts` | List payouts (address book) |
| 57 | POST | `/v1/externalEntities` | Create external entity |
| 58 | GET | `/v1/externalEntities` | List external entities |
| 59 | GET | `/v1/externalEntities/{walletId}` | Get external entity |
| 60 | POST | `/v2/reserveManagement/dailyCustodyBalances` | Daily custody report |
| 61 | POST | `/v1/notifications/subscriptions` | Create subscription |
| 62 | GET | `/v1/notifications/subscriptions` | List subscriptions |
| 63 | DELETE | `/v1/notifications/subscriptions/{id}` | Delete subscription |
