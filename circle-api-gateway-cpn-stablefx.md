# Circle Developer API Reference - Group 4: Gateway, CPN, StableFX, xReserve

> Comprehensive summary of 65 API reference pages covering Circle's Gateway, Circle Payments Network (CPN), StableFX, and xReserve services.

---

## Table of Contents

1. [Gateway API (7 endpoints)](#1-gateway-api)
2. [CPN Common API (7 endpoints)](#2-cpn-common-api)
3. [CPN Platform API (19 endpoints)](#3-cpn-platform-api)
4. [StableFX API (25 endpoints)](#4-stablefx-api)
5. [xReserve API (7 endpoints)](#5-xreserve-api)

---

## 1. Gateway API

**Base URLs:**
- Testnet: `https://gateway-api-testnet.circle.com`
- Production: `https://gateway-api.circle.com`

**Authentication:** No authentication required (security: []) for all Gateway endpoints.

**Supported Domains (13 chains):**

| Domain ID | Chain |
|-----------|-------|
| 0 | Ethereum |
| 1 | Avalanche |
| 2 | Optimism (OP) |
| 3 | Arbitrum |
| 5 | Solana |
| 6 | Base |
| 7 | Polygon PoS |
| 10 | Unichain |
| 13 | Sonic |
| 14 | World Chain |
| 16 | Sei |
| 19 | HyperEVM |
| 26 | Arc |

**Supported Token:** USDC only.

---

### 1.1 Get Gateway Info

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/info` |
| **Parameters** | None |
| **Auth** | None |

**Response (200):**
```json
{
  "version": 1,
  "domains": [
    {
      "chain": "Ethereum",
      "network": "Mainnet",
      "domain": 0,
      "walletContract": {
        "address": "0x...",
        "supportedTokens": ["USDC"]
      },
      "minterContract": {
        "address": "0x...",
        "supportedTokens": ["USDC"]
      },
      "processedHeight": "12345678",
      "burnIntentExpirationHeight": "12345600"
    }
  ]
}
```

**Key Notes:**
- Returns real-time block height data per network.
- Developers should add buffer to `burnIntentExpirationHeight` to prevent failures.

---

### 1.2 Estimate Transfer Fees

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/estimate` |
| **Auth** | None |

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `maxAttestationSize` | integer (min: 1) | No | Max allowed attestation size in bytes |
| `enableForwarder` | boolean (default: false) | No | Enable forwarding service for fee estimation |

**Request Body:** Array of partial burn intents or partial burn intent sets (min 1 item).

Each partial burn intent contains:
- `spec` (TransferSpec): version, domains, contracts, tokens, amounts, salt
- `maxBlockHeight` (string, optional): Block height validity limit
- `maxFee` (string, optional): Maximum acceptable fee

**Response (200):**
```json
{
  "body": [{ "burnIntent or burnIntentSet with calculated values" }],
  "fees": {
    "total": "string",
    "token": "USDC",
    "perIntent": [
      { "transferSpecHash": "...", "domain": 0, "baseFee": "...", "transferFee": "..." }
    ],
    "forwardingFee": "string (optional)"
  }
}
```

**Key Notes:**
- Estimates fees and expiration blocks without execution.
- EVM source domains support intent sets; single intents work across all domains.

---

### 1.3 Create Transfer Attestation

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/transfer` |
| **Auth** | None |

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `maxAttestationSize` | integer (min: 1) | No | Max attestation size in bytes; rejects with 400 if exceeded |
| `enableForwarder` | boolean (default: false) | No | Enable automatic mint submission forwarding |

**Request Body:** Array of SignedBurnIntent or SignedBurnIntentSet objects (min 1).
- `burnIntent` object + hex `signature`
- For EVM: `SignedBurnIntentSet` (array of burnIntents + signature)

**Response (201):**
```json
{
  "transferId": "UUID",
  "attestation": "0x...",
  "signature": "0x...",
  "fees": {
    "total": "...",
    "perIntent": [...],
    "forwardingFee": "..."
  },
  "expirationBlock": "string"
}
```

**Key Notes:**
- Attestation should be passed to the minter contract.
- Optional forwarding service handles automated minting.
- Fees aggregated across multiple burn intents.

---

### 1.4 Get Transfer by ID

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/transfer/{id}` |
| **Auth** | None |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `id` | UUID string | Yes | Unique transfer identifier |

**Response (200) - TransferDetailsResponse:**
- `destinationDomain` (domain enum)
- `status`: `pending` | `confirmed` | `finalized` | `failed` | `expired`
- `burnIntents` (array of BurnIntentSummary)
- `transactionHash` (only when confirmed/finalized/failed)
- `forwardingDetails` (forwardingEnabled, failureReason)
- `fees` (FeeSummary)
- `attestation` (payload, signature, expirationBlock -- when forwarding disabled)

**Key Notes:**
- Transaction hash only appears in non-pending statuses.
- Attestation data provided for manual minting when forwarding is disabled.

---

### 1.5 Get TransferSpec

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/transferSpec/{transferSpecHash}` |
| **Auth** | None |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `transferSpecHash` | Bytes32 (0x + 64 hex) | Yes | keccak256 hash of the TransferSpec |

**Response (200) - TransferSpecResponse:**
- `version` (integer)
- `sourceDomain`, `destinationDomain` (integer)
- `sourceContract`, `destinationContract` (32-byte padded hex)
- `sourceToken`, `destinationToken` (32-byte padded hex)
- `sourceDepositor`, `destinationRecipient` (32-byte padded hex)
- `sourceSigner`, `destinationCaller` (32-byte padded hex, zero = unrestricted)
- `value` (string), `salt` (string), `hookData` (string)

---

### 1.6 Get Pending Deposits

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/deposits` |
| **Auth** | None |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `token` | enum | Yes | Currently "USDC" only |
| `sources` | array | Yes | List of depositor address + optional domain |

**Response (200):**
```json
{
  "token": "USDC",
  "deposits": [
    {
      "depositor": "0x...",
      "domain": 0,
      "transactionHash": "0x...",
      "amount": "100.00",
      "status": "pending",
      "blockHeight": "...",
      "blockHash": "0x...",
      "blockTimestamp": "2025-01-01T00:00:00Z"
    }
  ]
}
```

---

### 1.7 Get Token Balances

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/balances` |
| **Auth** | None |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `token` | enum | Yes | "USDC" |
| `sources` | array | Yes | depositor address + optional domain |

**Response (200):**
```json
{
  "token": "USDC",
  "balances": [
    { "domain": 0, "depositor": "0x...", "balance": "1000.00" }
  ]
}
```

**Key Notes:** Omitting domain retrieves balances from all valid domains.

---

## 2. CPN Common API

**Base URL:** `https://api.circle.com`

**Authentication:** Bearer Token (format: `PREFIX:ID:SECRET`) -- required for all CPN endpoints except Ping.

---

### 2.1 Ping (Health Check)

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /ping` |
| **Auth** | None |

**Response (200):**
```json
{ "message": "pong" }
```

**Error:** 429 (Rate limit exceeded / TooManyRequests)

---

### 2.2 Create Webhook Subscription

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v2/cpn/notifications/subscriptions` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `endpoint` | string | Yes | Publicly accessible HTTPS URL; must respond 2XX to POST |
| `notificationTypes` | array | No | Event types or wildcards (`*`, `cpn.payment.*`). Omit for all |

**Header:** `X-Request-Id` (optional, for tracking)

**Response (200):**
```json
{
  "data": {
    "id": "UUID",
    "name": "...",
    "endpoint": "https://...",
    "enabled": true,
    "createDate": "...",
    "updateDate": "...",
    "notificationTypes": ["cpn.payment.*"],
    "restricted": true
  }
}
```

---

### 2.3 Get All Subscriptions

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v2/cpn/notifications/subscriptions` |
| **Auth** | Bearer Token |

**Parameters:** None

**Response (200):** Array of Subscription objects (same fields as create response).

---

### 2.4 Get Subscription by ID

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v2/cpn/notifications/subscriptions/{id}` |
| **Auth** | Bearer Token |

**Parameters:**

| Name | Type | Required |
|------|------|----------|
| `id` | UUID | Yes |

**Response (200):** Single Subscription object.

---

### 2.5 Update Subscription

| Field | Value |
|-------|-------|
| **Endpoint** | `PATCH /v2/cpn/notifications/subscriptions/{id}` |
| **Auth** | Bearer Token |

**Path:** `id` (UUID, required)

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `name` | string | Yes | Subscription name |
| `enabled` | boolean | Yes | Active status |

**Response (200):** Updated Subscription object.

---

### 2.6 Delete Subscription

| Field | Value |
|-------|-------|
| **Endpoint** | `DELETE /v2/cpn/notifications/subscriptions/{id}` |
| **Auth** | Bearer Token |

**Parameters:** `id` (UUID, required)

**Response:** 204 No Content (empty body).

---

### 2.7 Get Notification Signature Public Key

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v2/cpn/notifications/publicKey/{id}` |
| **Auth** | Bearer Token |

**Parameters:** `id` (UUID, required -- from webhook header `X-Circle-Key-Id`)

**Response (200):**
```json
{
  "data": {
    "id": "UUID",
    "algorithm": "ECDSA_SHA_256",
    "publicKey": "base64-encoded-key",
    "createDate": "..."
  }
}
```

**Key Notes:**
- Webhook headers contain `X-Circle-Signature` (digital signature) and `X-Circle-Key-Id`.
- Use this endpoint to verify webhook authenticity.

---

## 3. CPN Platform API

**Base URL:** `https://api.circle.com`

**Authentication:** Bearer Token (`PREFIX:ID:SECRET`) -- required for all endpoints.

**Supported Blockchains:** SOL, MATIC, ETH, SOL-DEVNET, MATIC-AMOY, ETH-SEPOLIA

**Supported Crypto:** USDC only

**Supported Fiat:** BRL, CNY, COP, EUR, GBP, HKD, INR, MXN, NGN, USD

---

### 3.1 Get Payment Configurations Overview

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/configurations/overview` |
| **Auth** | Bearer Token |

**Parameters:** None

**Response (200):**
```json
{
  "data": {
    "destinationCountries": ["AT", "BE", "BR", "US", ...],
    "destinationCurrencies": ["BRL", "EUR", "GBP", "USD", ...],
    "paymentMethodTypes": ["PIX", "SEPA", "WIRE", "CHATS", "FAST", "FPS", ...],
    "sourceCurrencies": ["USDC"],
    "blockchains": ["SOL", "MATIC", "ETH", ...]
  }
}
```

---

### 3.2 List Payment Routes

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/configurations/routes` |
| **Auth** | Bearer Token |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sourceCurrency` | enum | Yes | Crypto token (USDC) |
| `destinationCountry` | enum | Yes | ISO 3166-1 alpha-2 |
| `transactionVersion` | enum | No | VERSION_1 (default) or VERSION_2 |
| `destinationCurrency` | enum | No | Filter by fiat currency |
| `paymentMethodType` | enum | No | Filter by payment method |
| `blockchain` | enum | No | Filter by blockchain |

**Response (200):**
```json
{
  "data": [
    {
      "destinationCurrency": "EUR",
      "paymentMethodType": "SEPA",
      "blockchain": "ETH",
      "cryptoLimit": { "min": "...", "max": "..." },
      "fiatLimit": { "min": "...", "max": "..." }
    }
  ]
}
```

**Key Notes:** Determines valid corridors and parameters for subsequent quote creation.

---

### 3.3 Create Quote

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/quotes` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `paymentMethodType` | enum | Yes | AANI, BANK-TRANSFER, CHATS, CIPS, FAST, FPS, PIX, SEPA, SPEI, WIRE, etc. |
| `senderCountry` | enum | Yes | ISO 3166-1 alpha-2 |
| `sourceAmount` | object | Yes | Crypto amount/currency; leave amount blank if using destinationAmount |
| `destinationCountry` | enum | Yes | ISO 3166-1 alpha-2 |
| `destinationAmount` | object | Yes | Fiat amount/currency; leave amount blank if using sourceAmount |
| `blockchain` | enum | Yes | SOL, MATIC, ETH, or testnet variants |
| `senderType` | enum | Yes | BUSINESS or INDIVIDUAL |
| `recipientType` | enum | Yes | BUSINESS or INDIVIDUAL |
| `quoteOptions` | object | No | Whether sender and recipient are same entity |
| `transactionVersion` | enum | No | VERSION_1 (default) or VERSION_2 |

**Response (201):** Array of quote objects containing:
- `id`, `type` ("quote"), `paymentMethodType`, `blockchain`
- `senderCountry`, `destinationCountry`
- `createDate`, `quoteExpireDate`, `cryptoFundsSettlementExpireDate`
- `sourceAmount`, `destinationAmount`
- `fiatSettlementTime` (min/max with unit: MINUTES/HOURS/DAYS/WEEKS)
- `exchangeRate`, `rawExchangeRate`
- `fees` (breakdown array + totalAmount)
- `certificate` (BFI certificate with JWK public key)

**Key Notes:**
- Provide EITHER sourceAmount OR destinationAmount, not both with values.
- Quotes sorted by ascending sourceAmount (if based on destinationAmount) or descending destinationAmount.

---

### 3.4 Get Quote

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/quotes/{quoteId}` |
| **Auth** | Bearer Token |

**Parameters:** `quoteId` (UUID, required)

**Response (200):** Single Quote object (same fields as create response).

---

### 3.5 Get Payment Requirements

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/payments/requirements` |
| **Auth** | Bearer Token |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `quoteId` | UUID | Yes | Quote ID (cannot be reused across payments) |

**Response (200):**
```json
{
  "data": {
    "travelRule": [
      { "name": "field_name", "optional": false, "type": "TEXT|ADDRESS|FILE", "values": [...] }
    ],
    "beneficiaryAccount": [
      { "name": "field_name", "optional": false, "type": "TEXT", "values": [...] }
    ]
  }
}
```

**Key Notes:**
- Returns PII/compliance fields needed for the specific payment route.
- Travel rule fields: name, address, ID, DOB, nationality, etc.
- Beneficiary account fields: SWIFT, IBAN, routing numbers, etc.

---

### 3.6 Create Payment

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/payments` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `quoteId` | UUID | Yes | Previously created quote |
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `beneficiaryAccountData` | JWE | Yes | Encrypted beneficiary info (RFC 7516) |
| `travelRuleData` | JWE | Yes | Encrypted compliance data (RFC 7516) |
| `customerRefId` | string | Yes | Reference identifier |
| `beneficiaryRefId` | string | No | Beneficiary reference |
| `refCode` | string | No | Additional reference |
| `useCase` | enum | Yes | B2B, B2C, C2C, C2B |
| `reasonForPayment` | enum | Yes | PMT001-PMT030 codes |
| `senderAddress` | string | Yes | OFI wallet address |
| `blockchain` | enum | Yes | Blockchain network |
| `refundAddress` | string | Yes | Fallback wallet (same chain) |

**Response (201) - Payment object:**
- `id`, `quoteId`, `paymentMethodType`, `blockchain`
- `senderAddress`, `refundAddress`
- `status`: CREATED -> CRYPTO_FUNDS_PENDING -> FIAT_PAYMENT_INITIATED -> COMPLETED | FAILED
- `failureReason`, `failureCode` (PM##### format)
- `sourceAmount` (USDC), `destinationAmount` (fiat)
- `fees` (breakdown + total)
- `expireDate`, `createDate`
- `activeRfi`, `rfis`, `onChainTransactions`, `refunds`

**Key Notes:**
- Both beneficiaryAccountData and travelRuleData must be JWE-encrypted.
- Payment remains valid if onchain settlement occurs before settlementExpireDate.
- Status lifecycle: CREATED -> CRYPTO_FUNDS_PENDING -> FIAT_PAYMENT_INITIATED -> COMPLETED/FAILED

---

### 3.7 Get Payment

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/payments/{paymentId}` |
| **Auth** | Bearer Token |

**Parameters:** `paymentId` (UUID, required)

**Response (200):** Full Payment object (same fields as create response, plus `metadata`, `statusAddendum`).

---

### 3.8 Create Transaction (V1)

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/payments/{paymentId}/transactions` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `senderAccountType` | enum | Yes | Currently only EOA |

**Response (201) - Transaction object:**
- `id`, `paymentId`, `status` (CREATED/PENDING/BROADCASTED/COMPLETED/FAILED)
- `senderAddress`, `senderAccountType`, `blockchain`
- `amount` (USDC), `destinationAddress`
- `messageType`: EIP3009 or SOLANA
- `messageToBeSigned` (object)
- `estimatedFee` (EIP1559 or Solana fee structure)
- `expireDate`

**Key Notes:** Creates an *unsigned* onchain transaction. Must be signed separately before submission.

---

### 3.9 Create Transaction (V2)

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v2/cpn/payments/{paymentId}/transactions` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `metadata` | object | No | Key-value pairs (max 50 keys, 40 char keys, 500 char values) |

**Response (201):** Same as V1 but adds:
- `messageType` adds: PAYMENT_SETTLEMENT_CONTRACT_V1_0_PAYMENT_INTENT
- `metadata` field in response
- No `senderAccountType` or `estimatedFee` fields

---

### 3.10 Get Transaction (V1)

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/payments/{paymentId}/transactions/{transactionId}` |
| **Auth** | Bearer Token |

**Parameters:** `paymentId` (UUID), `transactionId` (UUID) -- both required.

**Response (200):** Full Transaction object (V1 schema).

---

### 3.11 Get Transaction (V2)

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v2/cpn/payments/{paymentId}/transactions/{transactionId}` |
| **Auth** | Bearer Token |

**Parameters:** `paymentId` (UUID), `transactionId` (UUID) -- both required.

**Response (200):** Full Transaction object (V2 schema, includes metadata).

---

### 3.12 Submit Transaction (V1)

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/payments/{paymentId}/transactions/{transactionId}/submit` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `signedTransaction` | string | Yes | Base64 (NEAR/Solana) or hex (EVM) encoded signed transaction |

**Response (201):** Updated Transaction object with `signedTransaction` and `transactionHash` populated.

**Key Notes:**
- Circle validates content and broadcasts to chain.
- Solana transactions include blockhash expiration (~1 minute).

---

### 3.13 Submit Transaction (V2)

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v2/cpn/payments/{paymentId}/transactions/{transactionId}/submit` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `signedTransaction` | string | Yes | Base64 (Solana) or hex (EVM) encoded |

**Response (200):** Updated Transaction V2 object.

**Key Notes:**
- Transaction must be in submittable status.
- Cannot resubmit after acceptance.
- Supports EIP-3009, Payment Settlement Contract (PSC), and Solana message types.

---

### 3.14 Accelerate Transaction

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/payments/{paymentId}/transactions/accelerate` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |

**Response (201):** New Transaction object with higher gas fees.

**Key Notes:**
- Used when broadcasted transactions remain unconfirmed (10+ minutes).
- Creates new transaction with identical parameters but higher gas.
- Prerequisites: No COMPLETED, CREATED, or PENDING transactions; only FAILED or BROADCASTED allowed.

---

### 3.15 Get Refund

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/payments/{paymentId}/refunds/{refundId}` |
| **Auth** | Bearer Token |

**Parameters:** `paymentId` (UUID), `refundId` (UUID) -- both required.

**Response (200) - Refund object:**
- `id`, `paymentId`
- `status`: CREATED | COMPLETED | FAILED
- `amount` (nullable when CREATED)
- `transactionHash` (nullable when CREATED)
- `refundAddress`, `blockchain`, `createDate`

---

### 3.16 Get RFI (Request for Information)

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/cpn/payments/{paymentId}/rfis/{rfiId}` |
| **Auth** | Bearer Token |

**Parameters:** `paymentId` (UUID), `rfiId` (UUID) -- both required.

**Response (200) - RFI object:**
- `id`, `paymentId`
- `status`: INFORMATION_REQUIRED | IN_REVIEW | APPROVED | REJECTED
- `level`: LEVEL_1 | LEVEL_2 | LEVEL_3
- `expireDate`
- `certificate` (id, certPem, domain, jwk -- BFI public key for encryption)
- `fieldRequirements` (version + JSON schema)
- `fileRequirements` (array: FORMATION_DOCUMENT, ORG_STRUCTURE, INVOICE, etc.)

**Key Notes:**
- Failure to respond before expiration results in payment failure.
- OFI must encrypt requested data using BFI's public certificate before sending.

---

### 3.17 Submit RFI

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/payments/{paymentId}/rfis/{rfiId}/submit` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `rfi` | object | Yes | Contains `version` (integer) and encrypted data (JWE format) |

**Response (200):** Updated RFI object.

**Key Notes:** Data must be encrypted as JWE. RFI must not be expired.

---

### 3.18 Upload RFI File

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/payments/{paymentId}/rfis/{rfiId}/files` |
| **Auth** | Bearer Token |
| **Content-Type** | `multipart/form-data` |

**Request Body (multipart):**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `fileMetadata` | object | Yes | fileName, fileType (PDF/DOCX/PNG/JPEG/CSV), fileKey (FORMATION_DOCUMENT, ORG_STRUCTURE, INVOICE, etc.) |
| `encryption` | object | Yes | encryptedAesKey (JWE), iv (base64 12-byte) |
| `encryptedFile` | binary | Yes | AES-128-GCM encrypted file data |

**Response:** 204 No Content (success).

**Key Notes:** File content requires AES-128-GCM encryption. AES key wrapped as JWE.

---

### 3.19 Create Support Ticket

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/cpn/supportTickets` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `paymentId` | UUID | Yes | Associated payment |
| `issueType` | enum | Yes | PAYMENT_SETTLEMENT_DELAY, PAYMENT_SENT_TO_WRONG_RECIPIENT, PAYMENT_AMOUNT_INCORRECT, REVERSE_FUNDS, OTHER |
| `description` | string | Yes | Detailed description |
| `email` | string | Yes | Contact email |
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `ccEmails` | array | No | CC email addresses |

**Response (201):** SupportTicket object (type, id, paymentId, issueType, description, caseRefId, ticketRefId, email, ccEmails, createDate).

---

## 4. StableFX API

**Base URL:** `https://api.circle.com`

**API Path Prefix:** `/v1/exchange/stablefx/`

**Authentication:** Bearer Token (`PREFIX:ID:SECRET`) -- required for all endpoints.

**Supported Currencies:** USDC, EURC

**Purpose:** Stablecoin-to-stablecoin exchange (e.g., USDC <-> EURC) using on-chain settlement with EIP-712 (Permit2) signatures.

---

### 4.1 Create Quote

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/exchange/stablefx/quotes` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `from` | CurrencyAmount | Yes | Source currency (USDC or EURC) + optional amount |
| `to` | CurrencyAmount | Yes | Target currency (USDC or EURC) + optional amount |
| `tenor` | enum | Yes | `instant`, `hourly`, or `daily` |

**CurrencyAmount:** `{ currency: "USDC"|"EURC", amount: "string (up to 6 decimals)" }`

**Response (200):**
```json
{
  "id": "UUID",
  "rate": 1.08,
  "from": { "currency": "USDC", "amount": "100.000000" },
  "to": { "currency": "EURC", "amount": "92.592593" },
  "createdAt": "...",
  "expiresAt": "...",
  "fee": "0.50",
  "collateral": "5.00"
}
```

**Key Notes:**
- Provide amount for `from` OR `to`, not both.
- Quotes expire at `expiresAt` timestamp.
- Fee in destination currency; collateral in source currency.

---

### 4.2 Create Trade

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/exchange/stablefx/trades` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `idempotencyKey` | UUID v4 | Yes | Exactly-once execution |
| `quoteId` | UUID | Yes | Previously created quote ID |

**Response (200) - Trade object:**
- `id` (UUID), `contractTradeId` (numeric string)
- `status`: pending | completed | confirmed | pending_settlement | taker_funded | maker_funded | breaching | breached
- `rate` (double), `from`, `to` (CurrencyAmount)
- `createDate`, `updateDate`, `quoteId`
- `settlementTransactionHash` (hex or null)

---

### 4.3 Get Trade by ID

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/exchange/stablefx/trades/{tradeId}` |
| **Auth** | Bearer Token |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `tradeId` | UUID | Yes | Trade ID |
| `type` | enum | Yes | `maker` or `taker` |

**Response (200) - TradeDetail object:** Same as Trade plus:
- `contractTransactions` object with:
  - `recordTrade` (status, txHash, errorDetails)
  - `takerDeliver` (status, txHash, errorDetails)
  - `makerDeliver` (status, txHash, errorDetails)

---

### 4.4 List Trades

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/exchange/stablefx/trades` |
| **Auth** | Bearer Token |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `type` | enum | Yes | `maker` or `taker` |
| `status` | enum | No | Filter by trade status |
| `pageSize` | integer | No | Default 50, max 200 |
| `from` | DateTime | No | Created since (inclusive) |
| `to` | DateTime | No | Created before (inclusive) |
| `settlementTransactionHash` | hex string | No | Filter by settlement tx hash |

**Response (200):**
```json
{
  "data": [{ "Trade objects..." }],
  "pagination": { "next": "...", "previous": "..." }
}
```

---

### 4.5 Get Trade Fee

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/exchange/stablefx/fees/{tradeId}` |
| **Auth** | Bearer Token |

**Parameters:** `tradeId` (UUID, required)

**Response (200):**
```json
{
  "tradeId": "UUID",
  "fee": { "currency": "USDC", "amount": "0.500000" }
}
```

---

### 4.6 Generate Trade Signature Data (Presign)

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/exchange/stablefx/signatures/presign/{traderType}/{tradeId}` |
| **Auth** | Bearer Token |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `traderType` | enum | Yes | `maker` or `taker` |
| `tradeId` | UUID | Yes | Trade ID |
| `recipientAddress` | string | No | Required when traderType is `taker` |

**Response (200):** EIP-712 `typedData` object containing:
- `domain` (Permit2 EIP-712 domain separator: name, chainId, verifyingContract)
- `types` (type definitions)
- `primaryType`: "PermitWitnessTransferFrom"
- `message` (permitted token, spender, nonce, deadline, witness with trade consideration)

**Key Notes:** Returns the EIP-712 payload the trader must sign for a CPS trade.

---

### 4.7 Register Trade Signature

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/exchange/stablefx/signatures` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `tradeId` | UUID | Yes | Trade identifier |
| `type` | enum | Yes | `maker` or `taker` |
| `address` | string | Yes | Wallet address that signed |
| `details` | TradePermit2Message | Yes | EIP-712 payload with permit2 data |
| `signature` | string | Yes | Generated signature |

**Response (200):**
```json
{
  "tradeId": "UUID",
  "type": "maker",
  "createDate": "...",
  "updateDate": "..."
}
```

**Key Notes:** Registers a signed EIP-712 payload confirming trade intent.

---

### 4.8 Generate Funding Presign Data

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/exchange/stablefx/signatures/funding/presign` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `contractTradeIds` | array[string] | Yes | List of contract trade IDs (min 1) |
| `type` | enum | Yes | `maker` or `taker` |
| `fundingMode` | enum | No | `gross` (default) or `net` |

**Response (200):** EIP-712 `typedData` for Permit2 funding signatures.
- Discriminated by `primaryType`:
  - `PermitWitnessTransferFrom` (single transfer)
  - `PermitWitnessBatchTransferFrom` (batch transfer)

---

### 4.9 Fund Trades

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/exchange/stablefx/fund` |
| **Auth** | Bearer Token |

**Request Body:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `type` | enum | Yes | `maker` or `taker` |
| `signature` | string | Yes | Permit2 signature (hex) |
| `fundingMode` | enum | No | `gross` (default) or `net` |
| `permit2` | object | Yes | Permit2 message (single or batch) |

**Permit2 Object (Single Trade):**
```json
{
  "permitted": { "token": "0x...", "amount": "1000000" },
  "spender": "0x...",
  "nonce": "123",
  "deadline": "1700000000",
  "witness": { "id": "trade-id-string" }
}
```

**Permit2 Object (Batch Trade):**
```json
{
  "permitted": [{ "token": "0x...", "amount": "..." }, ...],
  "spender": "0x...",
  "nonce": "...",
  "deadline": "...",
  "witness": { "ids": ["trade-id-1", "trade-id-2"] }
}
```

**Response:** 200 (success), 400 (validation error).

---

### 4.10 Webhook Subscriptions (StableFX-specific)

StableFX has its own webhook subscription management at `/v2/stablefx/notifications/...`:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `POST /v2/stablefx/notifications/subscriptions` | POST | Create subscription |
| `GET /v2/stablefx/notifications/subscriptions` | GET | List all subscriptions |
| `GET /v2/stablefx/notifications/subscriptions/{id}` | GET | Get subscription by ID |
| `PATCH /v2/stablefx/notifications/subscriptions/{id}` | PATCH | Update subscription |
| `DELETE /v2/stablefx/notifications/subscriptions/{id}` | DELETE | Delete subscription |
| `GET /v2/stablefx/notifications/publicKey/{id}` | GET | Get webhook signature verification key |

All follow the same schema as CPN Common webhook subscriptions but with `stablefx.trade.*` notification type patterns.

---

### 4.11 Webhook Event Types (StableFX)

The following webhook events are defined (delivered as POST to subscribed endpoints). Documentation for these provides only event descriptions without detailed payload schemas:

| Event | Description |
|-------|-------------|
| `tradeConfirmed` | Trade confirmed by the exchange |
| `tradePendingSettlement` | Trade confirmed onchain, awaiting funding from taker and maker |
| `tradeTakerFunded` | Taker's fund delivery transaction confirmed onchain |
| `tradeMakerFunded` | Maker's fund delivery transaction confirmed onchain |
| `tradeCompleted` | Both maker and taker funded; trade fully settled |
| `tradeFailed` | Trade has failed |
| `tradeBreached` | Trade breached (counterparty failed to fund in time) |
| `contractRecordTradeFailed` | Initial onchain recording of trade was unsuccessful |
| `contractTakerDeliverFailed` | Taker's deliver transaction failed to confirm onchain |
| `contractMakerDeliverFailed` | Maker's deliver transaction failed to confirm onchain |

---

### StableFX Trade Lifecycle Summary

```
Quote -> Create Trade -> [pending]
  -> Register Signatures (maker + taker)
  -> [confirmed] (onchain)
  -> [pending_settlement]
  -> Generate Funding Presign -> Fund (taker) -> [taker_funded]
  -> Generate Funding Presign -> Fund (maker) -> [maker_funded]
  -> [completed] (fully settled)

Failure paths:
  -> [breaching] -> [breached] (counterparty timeout)
  -> [failed] (various errors)
```

---

## 5. xReserve API

**Base URLs:**
- Testnet: `https://xreserve-api-testnet.circle.com`
- Production: `https://xreserve-api.circle.com`

**Authentication:** No authentication required (security: []) for all xReserve endpoints.

**Purpose:** Enables transfers between USDC and USDC-backed tokens across supported blockchain networks.

---

### 5.1 Get Domain Information

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/info` |
| **Auth** | None |

**Parameters:** None

**Response (200):**
```json
{
  "sourceDomains": [
    {
      "chain": "Ethereum",
      "network": "Sepolia",
      "domain": 0,
      "contractAddress": "0x...",
      "tokens": ["USDC"]
    }
  ],
  "remoteDomains": [
    {
      "chain": "...",
      "network": "...",
      "domain": 10001,
      "tokens": [
        {
          "remoteToken": "USDC",
          "remoteTokenIdentifier": "0x...",
          "associatedNativeToken": "USDC"
        }
      ]
    }
  ]
}
```

---

### 5.2 Get Token Balances

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/balances/{remoteDomain}` |
| **Auth** | None |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `remoteDomain` | integer | Yes | Remote domain ID (min: 0) |
| `token` | string | No | Filter by token symbol (e.g., "USDC") |

**Response (200):**
```json
{
  "balances": [
    { "token": "USDC", "balance": "1000000.00" }
  ]
}
```

**Key Notes:** Returns expected token balances based on deposit amounts made into xReserve.

---

### 5.3 Get Attestation

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/attestations/{depositMessageHash}` |
| **Auth** | None |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `depositMessageHash` | string | Yes | 32-byte hex (0x + 64 hex chars) |

**Response (200):**
```json
{
  "attestation": {
    "payload": "0x...",
    "messageHash": "0x...",
    "attestation": "0x..."
  }
}
```

---

### 5.4 List Attestations

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/remote-domains/{remoteDomain}/attestations` |
| **Auth** | None |

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `remoteDomain` | integer | Yes | Remote domain ID |
| `pageSize` | integer | No | 1-1000 results per page |
| `pageAfter` | string | No | Base64 cursor for forward pagination |
| `pageBefore` | string | No | Base64 cursor for backward pagination |
| `from` | DateTime | No | ISO 8601 start timestamp |
| `to` | DateTime | No | ISO 8601 end timestamp |

**Response (200):**
```json
{
  "attestations": [
    {
      "payload": "0x...",
      "messageHash": "0x...",
      "attestation": "0x..."
    }
  ]
}
```

**Headers:** `Link` header for cursor-based pagination (self, first, next, prev).

---

### 5.5 Prepare Withdrawal

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/prepare-withdrawal` |
| **Auth** | None |

**Request Body:**

`batches` array containing `PrepareBurnIntentInput` objects:

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `token` | enum | Yes | "USDC" only |
| `valueExcludingFees` | string | No* | Amount excluding fees (mutually exclusive with valueIncludingFees) |
| `valueIncludingFees` | string | No* | Amount including fees (mutually exclusive with valueExcludingFees) |
| `remoteDomain` | integer | Yes | Remote domain ID (must differ from finalDestinationDomain) |
| `remoteDepositor` | string | Yes | 32-byte hex address that initiated withdrawal on remote chain |
| `finalDestinationDomain` | integer | Yes | Destination domain (CCTP or remote) |
| `finalDestinationRecipient` | string | Yes | 32-byte hex recipient address |
| `finalDestinationCaller` | string | No | Authorized caller (validates identity if present) |
| `salt` | string | No | Uniqueness salt (random if omitted) |
| `useCircleForwarding` | boolean | Yes | Enable Circle transaction forwarding |
| `forwardingOptions` | object | No | maxFee, hookData, usesFastFinality |

**Response (200):**
```json
{
  "batches": [
    {
      "burnIntents": [{ "maxBlockHeight": "...", "maxFee": "...", "spec": { "TransferSpec..." } }],
      "encoded": "0x...",
      "messageHashToSign": "0x..."
    }
  ]
}
```

**Key Notes:**
- Encodes forwarding call data with calculated transfer amounts.
- Generates maxBlockHeight and maxFee with safety buffers.
- Result must be submitted to `/withdraw` with signatures.

---

### 5.6 Submit Withdrawal

| Field | Value |
|-------|-------|
| **Endpoint** | `POST /v1/withdraw` |
| **Auth** | None |

**Request Body:**

`batches` array (1-5 items), each `WithdrawBatch`:

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `burnIntents` | array | Yes | 1-10 burn intents per batch |
| `burnSignatures` | array | Yes | Multiple signatures (min 2 for multi-sig) |
| `burnTxId` | string | Yes | Burn transaction hash on remote chain |
| `useCircleForwarding` | boolean | Yes | Enable Circle forwarding |

**BurnIntent fields:** `maxBlockHeight`, `maxFee`, `spec` (TransferSpec with all domain/contract/token/address fields + structuredHookData).

**Response (201):**
```json
[
  {
    "withdrawalId": "UUID",
    "burnTxId": "0x...",
    "status": "created",
    "attestationPayload": "0x...",
    "attestation": "0x...",
    "useCircleForwarding": true,
    "transactionHash": "0x...",
    "failureReason": null,
    "transferSpecHashes": ["0x..."]
  }
]
```

**Status values:** created -> verified -> confirmed -> finalized (terminal) | expired | failed

**Error 409:** Conflict when burnTxId already associated with active withdrawal.

**Key Notes:**
- Up to 5 batches per request, 1-10 burn intents per batch.
- Minimum 2 signatures per batch (multi-sig requirement).

---

### 5.7 Get Withdrawal Status

| Field | Value |
|-------|-------|
| **Endpoint** | `GET /v1/withdrawal/{withdrawalId}` |
| **Auth** | None |

**Parameters:** `withdrawalId` (UUID, required)

**Response (200) - WithdrawalResponse:**
- `withdrawalId` (UUID)
- `burnTxId` (hex)
- `status`: created | verified | confirmed | finalized | expired | failed
- `attestationPayload` (hex)
- `attestation` (hex)
- `useCircleForwarding` (boolean)
- `transactionHash` (hex, when forwarded)
- `failureReason` (string, when failed)
- `transferSpecHashes` (array of hex strings)

**Key Notes:** Terminal states: finalized, expired, failed.

---

## Cross-Service Comparison

| Feature | Gateway | CPN | StableFX | xReserve |
|---------|---------|-----|----------|----------|
| **Auth Required** | No | Yes (Bearer) | Yes (Bearer) | No |
| **Token Support** | USDC | USDC | USDC, EURC | USDC |
| **Purpose** | Cross-chain USDC transfer | Crypto-to-fiat payments | Stablecoin exchange | Cross-chain reserve transfers |
| **Webhook Support** | No | Yes (CPN common) | Yes (StableFX-specific) | No |
| **Chains** | 13 domains | ETH, SOL, MATIC | EVM (EIP-712) | Source + Remote domains |
| **Base URL** | gateway-api.circle.com | api.circle.com | api.circle.com | xreserve-api.circle.com |
| **Testnet** | gateway-api-testnet | Same (testnet blockchains) | Same | xreserve-api-testnet |

---

## Common Patterns

### Authentication Format (CPN & StableFX)
```
Authorization: Bearer PREFIX:ID:SECRET
```
All three components required.

### Idempotency
Many POST endpoints accept `idempotencyKey` (UUID v4) to ensure exactly-once execution. Reusing the same key returns the original response.

### Error Response Format
```json
{
  "code": 400,
  "message": "Human-readable error",
  "errors": [
    { "error": "error_type", "message": "detail", "location": "field_path" }
  ]
}
```

### Request Tracking
All CPN/StableFX responses include `X-Request-Id` header (UUID) for Circle support communication.

### Webhook Signature Verification
1. Extract `X-Circle-Key-Id` and `X-Circle-Signature` from webhook headers.
2. Call `GET /v2/{service}/notifications/publicKey/{keyId}` to get public key.
3. Verify signature using the returned algorithm (ECDSA_SHA_256) and public key.
