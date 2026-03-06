# Circle Developer Documentation - Group 7: Services
## Comprehensive Summary of 99 Pages

> Circle Mint, Circle Payments Network (CPN), StableFX, Compliance Engine, Paymaster

---

# Table of Contents

1. [Circle Mint](#1-circle-mint)
   - [Overview & Getting Started](#11-overview--getting-started)
   - [Authentication & API Keys](#12-authentication--api-keys)
   - [Environments (Sandbox / Production)](#13-environments-sandbox--production)
   - [SDKs & Postman](#14-sdks--postman)
   - [Core API Patterns](#15-core-api-patterns)
   - [Deposits (Blockchain & Wire)](#16-deposits)
   - [Withdrawals (Bank & Blockchain)](#17-withdrawals)
   - [Crypto Payments (Deposits API)](#18-crypto-payments-deposits-api)
   - [Crypto Payouts](#19-crypto-payouts)
   - [Crypto Refunds](#110-crypto-refunds)
   - [Cross-Currency Exchange (USDC/EURC & Fiat)](#111-cross-currency-exchange)
   - [Notifications & Webhooks](#112-notifications--webhooks)
   - [Travel Rule (Onchain)](#113-travel-rule-onchain)
   - [Virtual Account Numbers](#114-virtual-account-numbers)
   - [Blockchain Confirmations](#115-blockchain-confirmations)
   - [Supported Chains, Currencies & Countries](#116-supported-chains-currencies--countries)
   - [API Errors & Error Codes](#117-api-errors--error-codes)
   - [Data Models (Resources)](#118-data-models)
   - [Release Notes (2024-2025)](#119-release-notes)

2. [Circle Payments Network (CPN)](#2-circle-payments-network-cpn)
   - [Overview & Architecture](#21-overview--architecture)
   - [API Integration & Authentication](#22-api-integration--authentication)
   - [Quotes](#23-quotes)
   - [Payments](#24-payments)
   - [Transactions (V1 & V2)](#25-transactions-v1--v2)
   - [RFI (Request for Information)](#26-rfi-request-for-information)
   - [Encryption (JWE / Travel Rule)](#27-encryption-jwe--travel-rule)
   - [Webhooks](#28-webhooks)
   - [Payment Configuration & Routes](#29-payment-configuration--routes)
   - [Supported Blockchains & Contracts](#210-supported-blockchains--contracts)
   - [Wallet Provider Compatibility](#211-wallet-provider-compatibility)
   - [Compliance References](#212-compliance-references)
   - [Error Codes & Failure Reasons](#213-error-codes--failure-reasons)
   - [Supported Payment Methods](#214-supported-payment-methods)
   - [Testing (Magic Values)](#215-testing-magic-values)
   - [OFI Integration Quickstart](#216-ofi-integration-quickstart)
   - [CPN Release Notes 2025](#217-cpn-release-notes-2025)

3. [StableFX](#3-stablefx)
   - [Overview](#31-overview)
   - [Technical Architecture](#32-technical-architecture)
   - [Taker Quickstart](#33-taker-quickstart)
   - [Maker Quickstart](#34-maker-quickstart)
   - [Smart Contract (FxEscrow)](#35-smart-contract-fxescrow)
   - [Permit2 & USDC Allowance](#36-permit2--usdc-allowance)
   - [Trade States](#37-trade-states)
   - [Talos Integration (Makers)](#38-talos-integration-makers)
   - [Webhooks & Signature Verification](#39-webhooks--signature-verification)
   - [Testing (Magic Numbers)](#310-testing-magic-numbers)
   - [Supported Currencies & References](#311-supported-currencies--references)
   - [StableFX Release Notes 2025](#312-stablefx-release-notes-2025)

4. [Compliance Engine](#4-compliance-engine)
   - [Overview](#41-overview)
   - [Transaction Screening](#42-transaction-screening)
   - [Screening Quickstart](#43-screening-quickstart)
   - [Rule Management](#44-rule-management)
   - [Alert Management](#45-alert-management)
   - [Testing with Magic Values](#46-testing-with-magic-values)
   - [OFAC & W3S Compliance Requirements](#47-ofac--w3s-compliance-requirements)

5. [Paymaster](#5-paymaster)
   - [Overview](#51-overview)
   - [Addresses & Events](#52-addresses--events)
   - [Pay Gas Fees in USDC Quickstart](#53-pay-gas-fees-in-usdc-quickstart)

---

# 1. Circle Mint

## 1.1 Overview & Getting Started

**Circle Mint** is Circle's institutional-grade platform for accessing and redeeming USDC and EURC. It serves as the fastest, most cost-effective way for exchanges, institutional traders, wallet providers, banks, and consumer-app companies to manage stablecoin distribution.

### Key Capabilities
- **24/7 Operations**: Round-the-clock fund management and crypto wallet distribution
- **Fiat Conversion**: Automatic conversion between traditional money (from 200+ countries) and stablecoins
- **1:1 Redemption**: Direct dollar/euro redemption guarantee through Circle
- **Bank Account Integration**: Register and manage business banking connections
- **Multi-blockchain Support**: Transfer digital assets across 30+ blockchain networks
- **Instant Settlement**: Participating banks offer rapid settlement programs

### Core API Features
- Digital currency transfers (USDC/EURC) in/out of accounts
- Fiat-to-crypto and crypto-to-fiat conversion flows
- Business bank account registration and management

### Additional APIs (Extra Cost)
- **Crypto Deposits API**: Accept USDC/EURC payments via onchain transfers
- **Crypto Payouts API**: Send USDC/EURC to customers, vendors, suppliers
- **Cross-Currency Exchange API**: Bidirectional swaps between local fiat and USDC, plus USDC-to-EURC swaps

---

## 1.2 Authentication & API Keys

### Authentication Mechanism
All Circle API requests require Bearer token authentication via HTTP headers:

```
Authorization: Bearer YOUR_API_KEY
```

- **HTTPS is mandatory** for all requests
- Unauthenticated requests fail immediately
- API keys must be stored securely (never in client-side code or public repos)

### Testing Authentication
Verify API key setup with the configuration endpoint:

```bash
curl -H 'Accept: application/json' \
  -H "Authorization: Bearer ${YOUR_API_KEY}" \
  -X GET --url https://api-sandbox.circle.com/v1/configuration
```

**JavaScript SDK:**
```javascript
import { Circle, CircleEnvironments } from "@circle-fin/circle-sdk";
const circle = new Circle("<your-api-key>", CircleEnvironments.sandbox);
const configResp = await circle.management.getAccountConfig();
```

**Success Response**: Returns `masterWalletId` in configuration data.
**Failure (401)**: "Malformed authorization" or improperly encoded credentials.

### API Key Types
1. **Standard Keys**: Access across all subscribed Circle APIs
2. **Restricted Keys**: Limited access based on selected product roles

### Key Management
- Create via Developer Dashboard (Circle Mint > Developers > API Keys)
- Maximum **10 keys per environment**
- Specify name and optional **IP allowlist**
- Production: `https://app.circle.com/developer`
- Sandbox: `https://app-sandbox.circle.com/developer`
- Revoke by selecting delete and confirming with "DELETE" text entry

---

## 1.3 Environments (Sandbox / Production)

| Aspect | Sandbox | Production |
|--------|---------|------------|
| Base URL | `https://api-sandbox.circle.com` | `https://api.circle.com` |
| Signup | `https://app-sandbox.circle.com/signup` | Via Circle onboarding |
| Real money | No (simulated) | Yes |
| Blockchain | Test networks | Mainnets |
| API Restrictions | Unrestricted | Role-based (403 for unauthorized) |
| Settlement Speed | Minimal delays | Real-world delays |
| Fees | May not reflect contract | Per client agreement |

### Health Check
```bash
curl https://api-sandbox.circle.com/ping
# Response: {"message": "pong"}
```

### Transition Guide
- Replace sandbox URLs with production endpoints
- Production API keys managed by Circle (cannot regenerate from dashboard)
- IP allowlist enforcement in production
- Test settlement timeframes and fee structures before going live
- Unauthorized endpoint calls return **403** in production

---

## 1.4 SDKs & Postman

### SDKs (Open Source)
| Language | Installation | Status |
|----------|-------------|--------|
| TypeScript/Node.js | `npm install @circle-fin/circle-sdk` | Stable |
| Java | Maven dependency | Beta |
| Python | `pip install circle-sdk` | Beta |

**TypeScript Setup:**
```javascript
import { Circle, CircleEnvironments } from "@circle-fin/circle-sdk";
const circle = new Circle("<your-api-key>", CircleEnvironments.sandbox);
```

### Postman Collections
Available collections: API Overview, Core Functionality, Crypto Deposits API, Crypto Payouts API.

**Usage Methods:**
- **Fork**: Copy with link to parent (synchronized updates)
- **View**: Immediate API experimentation
- **Import**: Local copy without link

Configure `apiKey` variable in Postman environment settings.

---

## 1.5 Core API Patterns

### Idempotent Requests
Circle APIs support idempotent requests via `idempotencyKey` body parameter (UUID format). This enables safe retry logic when network failures occur.

**Supported Operations:**
- Payout creation, Wire bank account creation, Business transfer creation
- Deposit address creation, Recipient address creation
- Payment intent creation/expiration/refunds
- Address book recipient creation

### Pagination & Filtering

**Date Filtering:**
- `from`: Inclusive start (ISO 8601 UTC); defaults to all history
- `to`: Inclusive end (ISO 8601 UTC); defaults to current time

**Pagination:**
- Default and max page sizes per endpoint (configurable via `pageSize`)
- Cursor-based: `pageAfter` and `pageBefore` query parameters with UUID cursors
- Link relations: `self`, `first`, `next`, `prev` in HTTP headers
- Data sorted by `createDate DESC`

### API Logs
- Accessible via Developer Tab; retained for **7 days**
- Captures: HTTP status, path, request ID, user agent, idempotency key, origin, timestamp, request/response bodies
- Filters: search, date range, status (2xx/4xx/5xx), method, path
- **Sensitive data redacted** with `"[redacted]"`

---

## 1.6 Deposits

### Deposit via Blockchain Wallet

**Create Deposit Address:**
```
POST /v1/businessAccount/wallets/addresses/deposit
Body: { "currency": "USD", "chain": "ETH" }
```
- Returns unique address ID and blockchain address
- **One deposit address per blockchain** limit

**List Deposit Addresses:**
```
GET /v1/businessAccount/wallets/addresses/deposit
```

**Check Balance:**
```
GET /v1/businessAccount/balances
```
Returns available and unsettled fund balances.

**Critical Warning**: Confirm blockchain network matches the address to avoid fund loss.

### Deposit via Wire (Funds Transfer)

**Step 1 - Create Wire Bank Account:**
```
POST /v1/businessAccount/banks/wires
```
Required: billing details, bank address, account/routing numbers, idempotencyKey.
Returns: account ID, status (pending), tracking reference, fingerprint.

**Step 2 - Get Wire Instructions:**
```
GET /v1/businessAccount/banks/wires/{TRANSFER_ID}/instructions?currency=USD
```
Returns: tracking reference, beneficiary details, bank info (name, address, SWIFT, routing number, account number).

**Step 3 - Mock Wire Payment (Sandbox):**
```
POST /v1/mocks/payments/wire
Body: { "amount": {...}, "trackingRef": "...", "beneficiaryAccountNumber": "..." }
```
Processes in batches, up to 15 minutes.

---

## 1.7 Withdrawals

### Withdraw to Bank (Wire)

**Step 1**: Create wire bank account (same as deposit flow).

**Step 2 - Create Payout:**
```
POST /v1/businessAccount/payouts
Body: { "destination": { "type": "wire", "id": "..." }, "amount": { "currency": "USD", "amount": "100.00" }, "idempotencyKey": "..." }
```
Requires sufficient account balance.

**Step 3 - Monitor Status:**
```
GET /v1/businessAccount/payouts/{id}
```
Status: pending -> complete.

**Note**: Singapore customers require recipient verification via Circle Console UI.

### Withdraw via Blockchain Wallet

**Step 1 - Register Recipient Address:**
```
POST /v1/businessAccount/wallets/addresses/recipient
Body: { "chain": "ETH", "address": "0x...", "currency": "USD", "idempotencyKey": "...", "description": "..." }
```
Returns address ID. **Requires manual admin validation.**

**Step 2 - Transfer Funds:**
```
POST /v1/businessAccount/transfers
Body: { "destination": { "type": "verified_blockchain", "addressId": "..." }, "amount": { "currency": "USD", "amount": "100.00" }, "idempotencyKey": "..." }
```

**Step 3 - Monitor:**
```
GET /v1/businessAccount/transfers/{transferId}
```
Status: created -> running -> complete. Transaction hash available at running status.

**France/Singapore**: Must verify recipients via Circle Console; unverified transfers remain pending.

---

## 1.8 Crypto Payments (Deposits API)

### Payment Flow (4 Steps)
1. Create payment intent (amount, currency, settlement, chain)
2. Obtain blockchain deposit address
3. Customer initiates wallet transfer
4. Circle confirms onchain and settles

### Create Payment Intent
```
POST /v1/paymentIntents
Body: {
  "amount": { "amount": "3.14", "currency": "USD" },
  "settlementCurrency": "USD",
  "paymentMethods": [{ "type": "blockchain", "chain": "ETH" }],
  "idempotencyKey": "..."
}
```

### Retrieve Address
- **Webhook**: Async notification when `paymentMethods.address` populated
- **Polling**: `GET /v1/paymentIntents/{id}` until address appears

### Typed Message Signing (EIP-712)
```
GET /payment/presign?paymentIntentId=...&endUserAddress=...
```
Returns EIP-712 typed data with domain (contract, chain ID), message (from/to, value, nonce), fee quote.

Customer signs, then:
```
POST /payments/crypto
Body: { "protocolMetadata": { "signature": "..." }, "paymentIntentId": "...", "quoteId": "...", ... }
```

### SDK Usage
```javascript
const circle = new Circle(apiKey, CircleEnvironments.sandbox);
await circle.cryptoPaymentIntents.createPaymentIntent(requestBody);
await circle.cryptoPaymentIntents.getPaymentIntent(paymentIntentId);
await circle.payments.getPayment(paymentId);
```

**Special**: Memo-required chains (XLM, HBAR) include `addressTag` in `depositAddress` object.

---

## 1.9 Crypto Payouts

### Workflow: 3 Phases

**Phase 1 - Create Recipient:**
```
POST /v1/addressBook/recipients
Body: { "idempotencyKey": "...", "chain": "ETH", "address": "0x...", "metadata": { "email": "...", "nickname": "..." } }
```
Status flow: `pending` -> `inactive` (24h hold if delayed withdrawals enabled) -> `active`.

**Phase 2 - Create Payout:**
```
POST /v1/payouts
Body: {
  "idempotencyKey": "...",
  "source": { "type": "wallet", "id": "...", "identities": [...] },
  "destination": { "type": "address_book", "id": "..." },
  "amount": { "amount": "100.00", "currency": "USD" },
  "toAmount": { "currency": "USD" }
}
```

**Critical**: `source.identities` **required for payouts >= $3,000** (FinCEN Travel Rule).

**Phase 3 - Monitor:** Poll `GET /v1/payouts/{id}` or use webhooks. Status: `pending` -> `complete`.

Response includes: payout ID, amounts, fees, network fees, transaction hash (`externalRef`), timestamps.

---

## 1.10 Crypto Refunds

**Refund Window**: Full or partial refund within **30 days** of payment intent creation.

```
POST /v1/paymentIntents/{id}/refund
Body: {
  "idempotencyKey": "...",
  "destination": { "chain": "ETH", "address": "0x..." },
  "amount": { "currency": "USD" },
  "toAmount": { "amount": "100.00", "currency": "USD" }
}
```

**Key Rules:**
- Pending crypto payments block refund initiation
- Once refunded, payment intents reject new deposits
- Funds sent to refunded intents become unsupported (need manual intervention)
- Also available via Circle Account UI (3-step flow)

---

## 1.11 Cross-Currency Exchange

### Fiat-to-USDC Exchange

**Step 1 - Get Linked Fiat Accounts:**
```
GET /v1/businessAccount/banks/pix
```

**Step 2 - Register Fiat Account:**
```
PUT /v1/exchange/fxConfigs/accounts
Body: { "fiatAccountId": "...", "currency": "..." }
```

**Step 3 - Request Quote:**
```
POST /v1/exchange/quotes
Body: { "type": "tradable", "idempotencyKey": "...", "from": { "currency": "MXN", "amount": "1000" }, "to": { "currency": "USD" } }
```
**Quotes valid for 3 seconds.** Returns: quote ID, locked rate, expiry.

**Step 4 - Execute Trade:**
```
POST /v1/exchange/trades
Body: { "idempotencyKey": "...", "quoteId": "..." }
```
Status: pending. Settlement depends on fiat transfer receipt.

**Step 5 - Settlement:**
```
GET /v1/exchange/trades/settlements
GET /v1/exchange/trades/settlements/instructions/{currency}
```
Returns beneficiary details, SWIFT codes, routing. Must send fiat on specified payment rail (e.g., SPEI for MXN).

### USDC-to-EURC Swap

Same quote/trade flow but specifying `from.currency: "USD"` and `to.currency: "EUR"`.

---

## 1.12 Notifications & Webhooks

### Setup
1. Expose publicly accessible HTTPS endpoint (supports HEAD and POST)
2. Register subscription via Circle Mint Dashboard (API > Subscriptions)
3. Confirm by visiting `SubscribeURL` in the SubscriptionConfirmation message

**Confirmation Message Fields:** Type, MessageId, Token, TopicArn, SubscribeURL, Signature, SigningCertURL.

### Subscription Limits
- Sandbox: **3 concurrent**, removed after 30 days
- Production: **1 subscription**

### Notification Types
| Type | States |
|------|--------|
| Payout | completed, failed |
| Transfer | created, failed, completed |

### Payload Structure
```json
{
  "clientId": "UUID",
  "notificationType": "payouts",
  "version": 1,
  "payout": { ... }
}
```

### API Management
All subscription management also available programmatically via API endpoints.

---

## 1.13 Travel Rule (Onchain)

FinCEN Travel Rule requires originator KYC for crypto transfers **>= $3,000** on applicable blockchains.

### Applicable Chains
Algorand, Aptos, Arbitrum, Avalanche, Base, Celo, Ethereum, NEAR, OP Mainnet, Polygon, Ripple, Solana, Stellar.

### Requirements by Endpoint

| Endpoint | Requirement |
|----------|-------------|
| `POST /v1/transfers` (blockchain, >= $3K) | Originator identity required |
| `POST /v1/payouts` (address_book, >= $3K) | Originator identity required |
| `POST /v1/businessAccount/transfers` | Uses pre-filed company identity |

### Identity Schema
```json
{
  "type": "individual",  // or "business"
  "name": "string (max 1024)",
  "addresses": [{
    "line1": "...", "city": "...",
    "country": "US",  // ISO 3166-1 alpha-2
    "postalCode": "..."  // max 16
  }]
}
```

Query with `returnIdentities=true` to retrieve identity data (limits response to 5 items).

---

## 1.14 Virtual Account Numbers

**Purpose**: Unique bank account identifier per linked fiat account, enabling automatic attribution of incoming wires without tracking references.

**Benefits**: Reduced wire returns, improved UX, no manual reference requirements.

**API Impact**: `GET /v1/businessAccount/banks/wires/{id}/instructions` now returns VAN in `beneficiaryBank.accountNumber` instead of omnibus account number. No schema changes.

---

## 1.15 Blockchain Confirmations

Circle sets chain-specific confirmation thresholds. Transfers move from `running` to `completed` upon reaching required confirmations.

| Chain | Confirmations | Time |
|-------|---------------|------|
| Algorand, Aptos, Avalanche, HyperEVM, NEAR, Noble, Sei, Solana, Sonic, Stellar, Sui, XRPL | **1** | 400ms-5s |
| XDC | **3** | 6s |
| Monad | **4** | 1.6s |
| Polygon PoS | **50** | 2 min |
| Ethereum, Base, Codex, Ink, OP Mainnet, Unichain, World Chain | **12 ETH blocks** | 3-9 min |
| Celo, Plume | **65 ETH blocks** | 13-19 min |
| Arbitrum | **300** | 3 min |
| Linea, Starknet, ZKsync Era | **65 ETH blocks** | 5-32 hours |

**L2 Handling**: Circle waits for L1 block inclusion after L2 finalization.

**Hedera**: Uses hashgraph consensus, no confirmation count.

**Wallet-to-wallet transfers** (source type `wallet`) complete instantly.

---

## 1.16 Supported Chains, Currencies & Countries

### USDC: 31+ Blockchains
Ethereum, Solana, Sui, Aptos, NEAR, Arbitrum, Optimism, Base, Linea, ZKsync Era, Starknet, Stellar, Polkadot Asset Hub, Noble, Avalanche, Polygon, Celo, and more.

API Currency Code: `USD`. Each chain has a specific API Chain Code (e.g., `ETH`, `SOL`, `ARB`).

### EURC: 6 Chains
Ethereum, Solana, Avalanche, Base, Stellar, World Chain. API Currency Code: `EUR`.

### Special Chain Requirements
- **Noble (Cosmos)**: Only supports USDC from Noble. Must transfer back to Noble before Circle Mint deposit.
- **Polkadot Asset Hub**: Similar IBC/XCM restrictions.

### Countries
- Wire transfers: ~150+ territories across all continents
- **Exclusions**: Hawaii and New York (US)
- Exchange services: Limited to Canada and USA

---

## 1.17 API Errors & Error Codes

### Error Response Format
```json
{
  "code": 1,
  "message": "Malformed authorization",
  "errors": [{ "error": "required", "message": "...", "location": "metadata.email" }]
}
```

### Key Error Categories

| Code Range | Category |
|-----------|----------|
| 1-3 | Authentication/Authorization |
| 1077-1097 | Payment-specific |
| 1096-1108 | Address/Compliance |
| 1143-1144 | Checkout |
| 2003-2007 | Blockchain operations |
| 5000-5015 | Travel rule/Payout |
| 500000-500005 | Custody balance |

### Entity Error Codes
- **Payment**: `payment_failed`, `payment_fraud_detected`, `payment_denied`, etc.
- **Card**: `card_invalid`, `card_expired`, `card_cvv_invalid`, etc.
- **Bank**: `invalid_account_number`, `account_name_mismatch`, etc.
- **Payout**: `insufficient_funds`, `transaction_denied`, `transaction_returned`
- **Transfer**: `transfer_failed`, `transfer_denied`, `blockchain_error`

---

## 1.18 Data Models

### Payout Object
- Fields: UUID id, source wallet ID, destination, amount (Money), fees, toAmount, status
- Status: `pending` | `complete` | `failed`
- Error codes: `insufficient_funds`, `transaction_denied`, `transaction_failed`, `transaction_returned`
- Includes risk evaluation (decision + reason)

### Transfer Object
- Routes: blockchain->wallet, wallet->wallet, wallet->blockchain
- Fields: UUID id, optional txHash, amount (Money), status
- Status: `pending` | `complete` | `failed`

### Money Object
```json
{ "amount": "3.14", "currency": "USD" }
```
String format, two decimal places. Currencies: USD, EUR.

### Source/Destination Types
- `wire` (fiat), `blockchain` (chain + address), `wallet` (wallet ID)

---

## 1.19 Release Notes

### 2025 Highlights
- **13 new blockchain supports** for USDC (Aptos, Unichain, Linea, XRPL, World Chain, Codex, Sei, HyperEVM, XDC, Plume, Ink, Monad, Starknet)
- EURC added to World Chain
- **Removed APIs** (Apr 2025): Pull Crypto Payments, Cards, Payment Tokens, Settlements, Checkout Sessions
- **Renamed**: Payments API -> Crypto Deposits API; Payouts API -> Crypto Payouts API
- FLOW and TRX chain support removed

### 2024 Highlights
- CUBIX account type support (Oct 2024)
- USDC on Sui (Oct 2024)
- Net burn fee daily reports endpoint (Sep 2024)
- Accounts API deprecated and removed (Dec 2024)

---

# 2. Circle Payments Network (CPN)

## 2.1 Overview & Architecture

**CPN** is a next-generation cross-border payment infrastructure that reduces reliance on intermediaries while enhancing transparency, security, and speed. It uses onchain USDC settlement with fiat off-ramp to local currencies.

### Key Participants
- **OFI** (Originating Financial Institution): Initiates payments
- **BFI** (Beneficiary Financial Institution): Receives and distributes fiat
- **CPN Platform**: Orchestrates the flow

### Core Features
- Real-time FX quoting with rate locking
- Smart routing to optimal BFIs based on pricing/preferences
- End-to-end encryption of travel rule and beneficiary data
- Comprehensive API suite with real-time webhook monitoring
- Ticketing support for disputes and reversals

### Prerequisites for OFI Onboarding
- Access to USDC liquidity
- Custodial solution with blockchain signing/observation capability
- Established KYC and AML processes

### CPN Payment Workflow (5 Stages)
1. **Quote Generation**: OFI requests aggregated quotes from multiple BFIs
2. **Payment Creation**: OFI encrypts and submits payment with travel rule data; BFI approves
3. **Onchain Transaction**: OFI signs and submits USDC transfer; CPN broadcasts
4. **Fiat Settlement**: BFI initiates local currency transfer; notifications propagate
5. **Payment Management**: Status tracking via API and webhooks

---

## 2.2 API Integration & Authentication

**Base URL**: `https://api.circle.com/v1/cpn/`

### Authentication
```
Authorization: Bearer {YOUR_API_KEY}
Content-Type: application/json
```

- HTTPS mandatory; HTTP rejected
- Missing/incorrect credentials: `401 - Invalid Credentials`
- IP allowlist available via Circle representatives
- Periodic key rotation recommended

### Idempotency
All state-changing endpoints require `idempotencyKey` (UUID v4) in request body to prevent duplicate transactions.

---

## 2.3 Quotes

**Endpoint**: `POST /v1/cpn/quotes`

### Key Parameters
- `paymentMethodType`: Payment corridor (e.g., SPEI, PIX, WIRE)
- `sourceAmount` / `destinationAmount`: Currency pairs
- `blockchain`: Target network
- `senderType` / `recipientType`: INDIVIDUAL or BUSINESS
- `transactionVersion`: VERSION_1 or VERSION_2

### Bidirectional Querying
- **Source-based**: Customer has USDC, wants to pay in BRL
- **Destination-based**: Recipient needs exact BRL amount

### Response
Includes: quote ID (time-limited), exchange rate, fee breakdown (TAX_FEE, BFI_TRANSACTION_FEE, CIRCLE_SERVICE_FEE, BLOCKCHAIN_GAS_FEE for V2), settlement windows, and **certificate with JWK** for encryption.

---

## 2.4 Payments

### Concept
A payment encompasses the complete CPN flow: onchain transactions + RFI compliance checks. Initiated by locking a quote with recipient details.

### Payment States
| State | Description |
|-------|-------------|
| `CREATED` | Quote accepted, compliance ongoing (up to 1 business day) |
| `CRYPTO_FUNDS_PENDING` | Compliance cleared, awaiting OFI onchain transaction |
| `FIAT_PAYMENT_INITIATED` | BFI validated crypto, initiated fiat payout |
| `COMPLETED` | Fiat transfer finished |
| `FAILED` | Payment cannot complete |

### Create Payment
```
POST /v1/cpn/payments
Body: {
  "quoteId": "...",
  "beneficiaryAccountData": "[encrypted JWE]",
  "travelRuleData": "[encrypted JWE]",
  "senderAddress": "0x...",
  "blockchain": "ETH",
  "idempotencyKey": "...",
  "reasonForPayment": "PMT001",
  "customerRefId": "...",
  "useCase": "..."
}
```

### Payment Requirements Discovery
```
GET /payments/requirements?quoteId={QUOTE_ID}
```
Returns required fields for `travelRule` and `beneficiaryAccount` schemas.

### Payment Reference (`refCode`)
Three-tier metadata support:
- **Full**: Sender name + refCode on bank statement
- **Partial**: BFI concatenates `{refCode} + {Sender Name}`
- **Minimal**: `fiatNetworkPaymentRef` (rail-specific reference)

### Failure & Recovery
If fiat transfer fails, BFI automatically issues crypto refund. Payment ID becomes unusable; new workflow required.

### Reason Codes (30 total)
Categories: business operations (PMT001-PMT004), financial/banking (PMT006-007), compensation (PMT008, PMT013-014), commercial (PMT009, PMT017-018, PMT029), personal (PMT022-025), investment (PMT026-030).

---

## 2.5 Transactions (V1 & V2)

### Transaction Flow (5 Steps)
1. **Initiate**: OFI requests raw transaction data from CPN
2. **Sign**: OFI validates and signs with wallet
3. **Submit**: OFI submits signed transaction
4. **Verify & Broadcast**: CPN verifies and broadcasts to blockchain
5. **Notify**: CPN notifies OFI and BFI via webhooks

### V1 vs V2 Comparison

| Aspect | V1 | V2 |
|--------|----|----|
| Chains | EVM + Solana | EVM only |
| Gas | Native tokens required | **USDC (fixed at quote time)** |
| Signing (EVM) | EIP-3009 + raw tx | EIP-712 Permit2 |
| Nonce management | Manual | Automatic |
| Acceleration | Manual | Auto-acceleration by CPN |
| Settlement | 1-12 hours | 0-5 minutes |

### V2 Gas Abstraction
Gas fees are included in the `fees` field as fixed USDC at quote creation. During settlement, the payment smart contract withdraws this fee. Fee stays constant regardless of blockchain gas fluctuations.

### V2 Permit2 Integration
Uses Uniswap's universal token approval system. Requires:
1. USDC allowance granted to Permit2 contract (`0x000000000022D473030F116dDEE9F6B43aC78BA3`)
2. EIP-712 typed-data signature of `PermitWitnessTransferFrom`

### Create Transaction (V2)
```
POST /v2/cpn/payments/{paymentId}/transactions
```
Returns unsigned EIP-712 typed data with domain (Permit2 contract), token permissions, nonce, deadline, and witness (PaymentIntent).

### Sign & Submit (V2)
Sign the `messageToBeSigned` using EIP-712, then:
```
POST /v2/cpn/payments/{paymentId}/transactions/{transactionId}/submit
Body: { "signedTransaction": "0x..." }
```

### Solana Flow
- V1: Deserialize `messageToBeSigned`, sign with ed25519 keypair, serialize to base64
- V2: Decode `encodedMessageToBeSigned` (base64), `partialSign()` with keypair
- **One-minute expiration** after signing

### Transaction States
`CREATED` -> `PENDING` -> `BROADCASTED` -> `COMPLETED` (or `FAILED`)

### Wallet Nonce Management (V1 Only)
- New wallets start at nonce 0; sequential without gaps
- Centralized tracking recommended with thread-safe locking
- V2 handles nonces automatically

### USDC Allowance to Permit2 (V2 Prerequisite)

**Permit2 Address** (all EVM chains): `0x000000000022D473030F116dDEE9F6B43aC78BA3`

**USDC has 6 decimals**: $100 USDC = `100000000`

```javascript
// Using Circle Wallets
createContractExecutionTransaction({
  walletId: "...",
  contractAddress: USDC_CONTRACT_ADDRESS,
  abiFunctionSignature: "approve(address,uint256)",
  abiParameters: [PERMIT2_ADDRESS, APPROVAL_AMOUNT],
  idempotencyKey: "...",
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

---

## 2.6 RFI (Request for Information)

### Concept
BFI requests additional compliance data about sender or OFI at any payment stage. 3 escalating levels.

### RFI States
| State | Description |
|-------|-------------|
| `INFORMATION_REQUIRED` | OFI must submit data |
| `IN_REVIEW` | BFI reviewing submission |
| `APPROVED` | BFI accepts |
| `FAILED` | BFI rejects (terminal, no resubmission) |

### RFI Levels

**Individual:**
- Level 1: Address, Name, DOB, National ID, Source of Funds, Verification Method
- Level 2: + Nationality, Email, Phone, Occupation
- Level 3: + ID document type/copy, Proof of address, Supplementary docs

**Business:**
- Level 1: Name, Trade Name, National ID, Formation Date/Country, Entity/Industry Type, Address
- Level 2: + Authorized signatories, Beneficial ownership, Website, Email, Phone
- Level 3: + Formation docs, Address proof, Org structure, Payment invoices, Beneficial owner IDs

Beneficial owner threshold: **>= 25% ownership**.

### RFI Response Workflow
1. Retrieve RFI object (if async: `GET /api-reference/cpn/cpn-platform/get-rfi`)
2. Build JSON response from `fieldRequirements` schema
3. Validate against JSON Schema locally
4. Encrypt using ECDH-ES+A128KW with A128GCM
5. Submit encrypted data
6. Upload encrypted files via `multipart/form-data`

**Non-response consequence**: Payment automatically cancelled.

---

## 2.7 Encryption (JWE / Travel Rule)

### Framework
CPN uses JSON Web Encryption (JWE) for travel rule and beneficiary account data encryption. Keys shared in quote response.

### Required Parameters
- Key Agreement: `ECDH-ES+A128KW`
- Encryption Method: `A128GCM`

### Workflow

**Step 1 - Certificate from Quote:**
```json
{
  "certPem": "base64-encoded cert",
  "domain": "expected-domain",
  "jwk": { "kty": "EC", "crv": "...", "kid": "...", "x": "...", "y": "..." }
}
```

**Step 2 - Verify Certificate (Production):**
Check expiration, CA signature, common name, public key alignment.

**Step 3 - Build Payload:**
```json
[
  { "name": "FIELD_NAME", "value": "field_value" },
  { "name": "BENEFICIARY_ACCOUNT", "value": "..." }
]
```

**Step 4 - Encrypt:**
```java
JWEHeader header = new JWEHeader(JWEAlgorithm.ECDH_ES_A128KW, EncryptionMethod.A128GCM);
JWEObject jwe = new JWEObject(header, new Payload(jsonString));
jwe.encrypt(new ECDHEncrypter(recipientJWK));
String encrypted = jwe.serialize();
```

**Step 5 - Submit**: 200 response if properly encrypted.

### File Encryption (RFI)
Two-stage: files encrypted with AES-128-GCM, then AES key wrapped with JWE.

1. Generate 128-bit AES key
2. Generate 12-byte IV
3. Encrypt file with AES/GCM/NoPadding
4. Wrap AES key in JWE using BFI's public EC key
5. Submit via multipart/form-data: `fileMetadata` + `encryption` (encryptedAesKey, iv) + `encryptedFile`

**Critical**: Do not manually set Content-Type header; let HTTP client set it with boundary.

### JSON Schema Validation
CPN uses Draft 2020-12 JSON Schema. Recommended validation libraries:
- Java: `networknt/json-schema-validator`
- Python: `jsonschema`
- Node.js: `ajv`

---

## 2.8 Webhooks

### Setup
```
POST /v2/cpn/notifications/subscriptions
Body: {
  "endpoint": "https://...",
  "name": "My Subscription",
  "enabled": true,
  "notificationTypes": ["*"]
}
```

Endpoint requirements: publicly accessible, HTTPS, supports HEAD and POST.

### Webhook Events

**Payment Events (6):**
- `cpn.payment.cryptoFundsPending` - Waiting for funds deposit
- `cpn.payment.fiatPaymentInitiated` - BFI confirmed, fiat started
- `cpn.payment.completed` - Fiat sent, processing done
- `cpn.payment.failed` - Error/compliance rejection
- `cpn.payment.delayed` - Settlement delay
- `cpn.payment.inManualReview` - Under manual review

**RFI Events (4):**
- `cpn.rfi.informationRequired`, `cpn.rfi.inReview`, `cpn.rfi.approved`, `cpn.rfi.rejected`

**Transaction Events (3):**
- `cpn.transaction.broadcasted`, `cpn.transaction.completed`, `cpn.transaction.failed`

**Refund Events (3):**
- `cpn.refund.created`, `cpn.refund.failed`, `cpn.refund.completed`

### Payload Structure
```json
{
  "subscriptionId": "...",
  "notificationId": "...",
  "notificationType": "cpn.payment.completed",
  "notification": { ... },
  "timestamp": "ISO 8601",
  "version": 2
}
```

### Signature Verification
Headers: `X-Circle-Signature` (base64 signature), `X-Circle-Key-Id` (UUID).

```python
# Fetch public key
GET /v2/cpn/notifications/publicKey/{keyId}
# Returns: algorithm "ECDSA_SHA_256", base64 DER public key

# Verify
from cryptography.hazmat.primitives.asymmetric import ec
verifier = public_key.verifier(signature, ec.ECDSA(hashes.SHA256()))
verifier.update(payload_bytes)
verifier.verify()
```

**Cache public keys** to reduce API calls.

### Retry Policy
| Attempt | Testing | Production |
|---------|---------|-----------|
| Max retries | 6 | 11 |
| Schedule | 1s, 10s, 30s, 1m, 15m, 1h | 1s, 10s, 30s, 1m, 15m, 1h, 3h, 5h, 6h, 10h, 11h |

---

## 2.9 Payment Configuration & Routes

### Configuration Overview
```
GET /v1/cpn/v1/ofi/configurations/overview
```
Returns supported: source currencies, destination countries/currencies, payment methods, blockchains.

### Routes
```
GET /v1/cpn/v1/ofi/configurations/routes?sourceCurrency=USDC&destinationCountry=MX&transactionVersion=VERSION_2
```
Returns: destination currency, payment method, blockchain, fiat limits (min/max), crypto limits (min/max).

**V2 Note**: Chain-specific buffer added to crypto min limit to cover onchain fees.

---

## 2.10 Supported Blockchains & Contracts

### Supported Networks

| Environment | Chains |
|-------------|--------|
| Testnet | Ethereum Sepolia, Polygon Amoy, Solana Devnet |
| Mainnet | Ethereum, Polygon, Solana |

CPN is **chain-agnostic** and built for multichain flexibility.

### Contract Addresses
Settlement contract deployed at same address across networks:
`0x355e0a2a4B7563e0E00C90deD9Aa914c119Ee868`

Available on: Ethereum mainnet, Polygon PoS, Arc Testnet, Ethereum Sepolia, Polygon Amoy.

---

## 2.11 Wallet Provider Compatibility

CPN supports custom wallet providers (not limited to Circle Wallets).

### Requirements

| Feature | V1 | V2 |
|---------|----|----|
| EVM | EIP-712 + raw tx signing | EIP-712 only |
| Solana | Standard signing | Standard + partial signing + memo |
| Custodial | Must sign without user interaction | Same |

### Smart Contract Account (SCA) Support
Must implement:
- EIP-165 Standard Interface Detection
- EIP-1271 Standard Signature Validation (`isValidSignature`)

USDC implements EIP-3009 with EIP-7598 extension for both EOAs and SCAs.

---

## 2.12 Compliance References

### Supported Countries
249 countries/territories listed. **16 unsupported** (sanctioned): Afghanistan, Belarus, Central African Republic, Congo DR, Cuba, Guinea-Bissau, Iran, Iraq, North Korea, Laos, Libya, Mali, Myanmar, Russia, Somalia, South Sudan, Sudan, Syria, Ukraine, Venezuela, Yemen.

### Travel Rule Requirements
Data submitted at payment creation. Scope varies by geography.

**OFI Data**: Business name and address.
**Individual Fields**: Names, account numbers, addresses, DOB, nationality, national ID.
**Business Fields**: Legal entity names, formation dates/country, registration/tax IDs.

### Brazilian Tax ID Validation
**CNPJ** (14 digits): 8-digit base + 4-digit branch + 2 check digits (mod 11 algorithm).
**CPF** (11 digits): 9 base digits + 2 check digits (mod 11 algorithm).
Both reject all-identical-digit sequences.

---

## 2.13 Error Codes & Failure Reasons

### Common Errors
| Code | HTTP | Issue |
|------|------|-------|
| 4 | 401 | Invalid credentials |
| 3 | 403 | Insufficient permissions |
| 2 | 400 | Missing/malformed parameters |
| 2900000 | 400 | Testnet/mainnet mismatch |

### Quote Errors (290100-290102)
Amount exceeds limits, BFI unavailable, unsupported route.

### Payment Errors (290200-290208)
Invalid/expired/used quote, blockchain mismatch, sanctions hit, RFI pending/rejected, missing params.

### Transaction Errors (290300-290341)
Active tx exists, wrong state, expiration, insufficient balance/gas, nonce issues, signing mismatches, Permit2 problems.

### Payment Failure Reasons (10)
`TRAVEL_RULE_FAILED`, `BANK_VERIFICATION_FAILED`, `RFI_VERIFICATION_FAILED`, `EXISTING_RFI_PENDING`, `ONCHAIN_SETTLEMENT_FAILED`, `FIAT_SETTLEMENT_FAILED`, `COMPLIANCE_CHECK_FAILED`, `CANCELLED`, `PAYMENT_EXPIRED`, `OTHER`.

### Transaction Failure Reasons (13)
Expiration (4), nonce/state (4), gas/resource (4), execution (1).

### Payment Failure Codes (PM00001-PM09000)
Travel rule (PM01xxx), bank verification (PM02xxx), RFI (PM03xxx), pending RFI (PM04xxx), onchain settlement (PM05xxx), fiat settlement (PM06xxx), compliance (PM07xxx), cancellation (PM08xxx), expiry (PM09xxx).

---

## 2.14 Supported Payment Methods

### Settlement Categories
1. **Instant** (seconds to minutes): PIX, SPEI, IMPS, FPS, NEQUI, BANK-TRANSFER (Nigeria)
2. **Batched** (1-2 business days): WIRE, CHATS, SEPA

### Regional Matrix
| Region | Methods | Currencies |
|--------|---------|-----------|
| Brazil | PIX, WIRE | BRL, USD |
| Mexico | SPEI, WIRE | MXN, USD |
| India | IMPS, RTGS, NEFT | INR |
| Hong Kong | FPS, CHATS, WIRE | HKD, USD |
| Colombia | NEQUI, BANK-TRANSFER, WIRE | COP, USD |
| Nigeria | BANK-TRANSFER, WIRE | NGN, USD |
| US | FEDWIRE | USD |
| EU | WIRE (SEPA fallback) | USD, EUR |

Quote response includes `fiatSettlementTime` field.

---

## 2.15 Testing (Magic Values)

All case-sensitive. Use `ORIGINATOR_NAME` field in create payment:

| Magic Value | Behavior |
|-------------|----------|
| `Failed` | Synchronous FAILED status |
| `AsyncFailed` | CREATED then webhook -> FAILED |
| `AsyncSuccess` | CREATED then webhook -> CRYPTO_FUNDS_PENDING |
| `CreateRfi` | Level 1 RFI in sync response |
| `CreateRfiL2` / `CreateRfiL3` | Level 2/3 RFI |
| `AsyncRfi` | CREATED then webhook creates RFI |
| `Delayed` | Fiat settlement delay after FIAT_PAYMENT_INITIATED |
| `Expired` | CRYPTO_FUNDS_PENDING then FAILED (PAYMENT_EXPIRED) |
| `FailThenRefundWithCompleted` | Failed + completed refund |
| `FailThenRefundCreatedThenFailed` | Failed + failed refund |
| `FailThenRefundCreatedThenCompleted` | Failed + successful refund |

**RFI Magic Values** (`NAME` field):
- `InReview`: Stuck in IN_REVIEW
- `Rejected`: RFI rejection
- Any other value: approval

---

## 2.16 OFI Integration Quickstart

### Prerequisites
- CPN API key
- Circle Developer Account with wallet set, EOA wallet, testnet USDC + ETH
- Python with `jwcrypto`, `web3`, `eth_utils` libraries
- (V2) USDC allowance to Permit2

### Part 1: Request Quotes
```bash
POST /v1/cpn/quotes
Body: {
  "paymentMethodType": "SPEI",
  "sourceAmount": { "currency": "USDC", "amount": "100" },
  "destinationCountry": "MX",
  "blockchain": "ETH",
  "senderType": "INDIVIDUAL",
  "recipientType": "INDIVIDUAL",
  "transactionVersion": "VERSION_2"
}
```

### Part 2: Create Payment
1. Get requirements: `GET /payments/requirements?quoteId=...`
2. Encrypt travel rule + beneficiary data with JWE (ECDH-ES+A128KW)
3. Submit: `POST /payments` with encrypted data

### Part 3: Execute Transaction
1. Create: `POST /v2/cpn/payments/{paymentId}/transactions`
2. Sign EIP-712 typed data
3. Submit: `POST /v2/cpn/payments/{paymentId}/transactions/{transactionId}/submit`

CPN broadcasts, BFI confirms, fiat settlement via webhooks.

### Fee Categories
TAX_FEE, BFI_TRANSACTION_FEE, CIRCLE_SERVICE_FEE, BLOCKCHAIN_GAS_FEE (V2 only).

---

## 2.17 CPN Release Notes 2025

### August 2025
- Payment failure codes added
- `quoteOptions` field in POST /v1/cpn/quotes
- `failureCode` field in payment endpoints
- New error codes: 290207, 290302-290305

### July 2025
- Complete OFI documentation (19 new guides)
- 20 new CPN API endpoints (quotes, payments, transactions, RFIs, support tickets)

---

# 3. StableFX

## 3.1 Overview

**StableFX** is an institutional-grade stablecoin FX engine built on Arc blockchain that combines Request-for-Quote (RFQ) execution with onchain settlement. Launched November 2025.

### Target Users
Payment providers, fintechs, OTC desks, institutions.

### Supported Currencies
USDC and EURC (with plans for additional local stablecoin pairs).

### Key Features
- Aggregated liquidity from multiple providers via single API call
- 24/7 settlement with sub-second finality on Arc
- Smart contract escrow (payment-versus-payment)
- API-first (no direct smart contract interaction required)
- Minimum trade: 10 USDC

### Use Cases
- Cross-border payments with real-time conversion
- Institutional treasury rebalancing
- Embedded FX liquidity for platforms
- Remittance solutions (eliminating T+2 delays)

---

## 3.2 Technical Architecture

### Two Layers

1. **Execution Engine** (offchain): API managing RFQ distribution, validation, quote ranking, price discovery, trade execution, signature collection.

2. **Settlement Contract** (`FxEscrow`): Arc smart contract performing escrow and delivery. PvP mechanics ensure simultaneous settlement or complete failure.

### API
- **Base URL**: `https://api.circle.com/`
- **Key Types**: `TEST` (Arc testnet, mock data) or `LIVE` (Arc mainnet)
- Authentication: Bearer token

### Quote Parameters
- `from.currency` / `from.amount`, `to.currency` / `to.amount`
- `tenor`: `instant` (30 min), `hourly` (1 hour), `daily` (1 day)
- Response time: under 500ms

### Fee Logic
- **Buy currency specified**: Fee added to buy amount before Talos request
- **Sell currency specified**: Fee applied post-quote in sell currency

### Settlement Model
- Explicit taker funding, maker-triggered settlement
- Taker receives: buy currency minus taker fee
- Maker receives: sell currency minus maker fee
- Fee wallet receives: both fees
- 10-minute window for signatures after trade acceptance

### Wallet Requirements
- Arc-supported wallets only
- Must support Permit2 contract
- EIP-712 typed data signing capability
- Individual ownership (no omnibus)
- Same wallet throughout single trade

---

## 3.3 Taker Quickstart

### Part 1: Quote Generation
```bash
POST /v1/exchange/stablefx/quotes
Body: {
  "from": { "currency": "USDC", "amount": "1000" },
  "to": { "currency": "EURC" }
}
```
Response: quote ID, rate (e.g., 0.915), amounts, fee, expiry.

### Part 2: Trade Creation
```bash
POST /v1/exchange/stablefx/trades
Body: { "quoteId": "...", "idempotencyKey": "..." }
```
Returns trade ID, status `pending`, contractTradeId.

### Part 3: Trade Intent (Signature Registration)
```bash
# 3.1 Get presign data
GET /v1/exchange/stablefx/signatures/presign/taker/{tradeId}?recipientAddress=0x...

# 3.2 Sign EIP-712 typed data with wallet

# 3.3 Register signature
POST /v1/exchange/stablefx/signatures
Body: { "tradeId": "...", "type": "taker", "signerAddress": "0x...", "typedData": {...}, "signature": "0x..." }

# 3.4 Verify status -> pending_settlement
GET /v1/exchange/stablefx/trades/{tradeId}
```

### Part 4: Onchain Funding
```bash
# 4.1 Get funding presign data
POST /v1/exchange/stablefx/signatures/funding/presign
Body: { "contractTradeIds": ["..."], "type": "taker" }

# 4.2 Sign Permit2 typed data

# 4.3 Submit funding
POST /v1/exchange/stablefx/fund
Body: {
  "type": "taker",
  "trades": [{ "signature": "0x...", "permit2": { "permitted": { "token": "...", "amount": "..." }, "spender": "...", "nonce": "...", "deadline": "...", "witness": "..." } }]
}

# 4.4 Verify -> taker_funded
GET /v1/exchange/stablefx/trades/{tradeId}
```

Alternative: Submit funding transaction directly via web3 provider.

---

## 3.4 Maker Quickstart

### Part 1: Trade Discovery
```bash
GET /v1/exchange/stablefx/trades?type=maker&status=confirmed
```

### Part 2: Signature Registration
Same flow as taker but with `type: "maker"`.

### Part 3: Funding
Wait for taker to fund (status: `taker_funded`), then:
```bash
POST /v1/exchange/stablefx/signatures/funding/presign
Body: { "contractTradeIds": ["..."], "type": "maker" }
```
Sign and submit via fund endpoint or directly onchain.

Makers can use `calculateMakerNet()` to compute net positions across multiple trades.

---

## 3.5 Smart Contract (FxEscrow)

Deployed at: `0x867650F5eAe8df91445971f14d89fd84F0C9a9f8` (Arc testnet)

### Core Functions
| Function | Description |
|----------|------------|
| `recordTrade()` | Initialize trade onchain (both signatures required) |
| `takerDeliver()` | Single-trade quote currency transfer via Permit2 |
| `takerBatchDeliver()` | Batch taker deliveries |
| `makerDeliver()` | Single-trade base currency transfer |
| `makerBatchDeliver()` | Batch maker deliveries |
| `makerNetDeliver()` | Net settlement across positions |
| `breach()` | Mark trade as defaulted post-maturity |
| `calculateMakerNet()` | Compute aggregate token positions |

All delivery functions use Permit2 for signature-based transfers.

---

## 3.6 Permit2 & USDC Allowance

Permit2 address (all EVM chains): `0x000000000022D473030F116dDEE9F6B43aC78BA3`

USDC uses 6 decimals: $100 = 100,000,000 units.

```javascript
// Using Circle Wallets
createContractExecutionTransaction({
  walletId: CIRCLE_WALLET_ID,
  contractAddress: USDC_CONTRACT_ADDRESS,
  abiFunctionSignature: "approve(address,uint256)",
  abiParameters: [PERMIT2_CONTRACT_ADDRESS, APPROVAL_AMOUNT],
  idempotencyKey: uuid(),
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

---

## 3.7 Trade States

| State | Terminal? | Description |
|-------|-----------|-------------|
| `pending` | No | Initial; exchange provider processing |
| `confirmed` | No | Provider confirmed; awaiting signatures |
| `pending_settlement` | No | Recorded onchain; awaiting funding |
| `taker_funded` | No | Taker delivered; awaiting maker |
| `maker_funded` | No | Maker delivered; awaiting taker |
| `breaching` | No | Maturity passed; processing expiration |
| `breached` | Yes | Both parties missed deadline |
| `failed` | Yes | Expired before onchain recording |
| `complete` | Yes | Settlement successful |

---

## 3.8 Talos Integration (Makers)

StableFX uses the **Talos platform** for routing quote requests and trade orders to market makers.

### Quote Request Flow
```json
{
  "Symbol": "EURC-USDC",
  "QuoteReqID": "UUID",
  "OrderQty": "10",
  "Markets": ["keyrock", "cumberland"],
  "TransactTime": "ISO 8601"
}
```

1. Talos returns aggregated quotes from multiple makers
2. StableFX selects optimal quote
3. Taker creates trade via API using Talos RFQ ID
4. StableFX calls Talos orders API
5. Upon confirmation, trade recorded onchain

---

## 3.9 Webhooks & Signature Verification

### Setup
```bash
POST /v2/stablefx/notifications/subscriptions
Body: { "endpoint": "https://...", "notificationTypes": ["*"] }
```

### Signature Verification

**Step 1 - IP Allowlisting:**
- `3.230.111.7`, `3.90.127.28`, `35.169.154.32`, `54.88.227.75`
- Non-allowlisted: return 403

**Step 2 - Extract Headers:**
- `X-Circle-Signature`: base64 signature
- `X-Circle-Key-Id`: UUID

**Step 3 - Verify:**
```
GET /v2/stablefx/notifications/publicKey/{id}
```
Algorithm: ECDSA_SHA_256. Key: DER-encoded EC.

**Critical**: Read raw request body as string for verification. Parsing/re-serializing JSON changes key ordering and fails verification.

---

## 3.10 Testing (Magic Numbers)

| Amount | Effect |
|--------|--------|
| `23.66` | Opposite party signature won't be registered during recording |
| `23.67` | Opposite party funding won't be provided |

Use in `from.amount` or `to.amount` for takers; seeded in trades for makers.

---

## 3.11 Supported Currencies & References

Currently supported: **USDC** and **EURC** only. Plans for additional local stablecoin pairs.

---

## 3.12 StableFX Release Notes 2025

### November 2025 Launch
- 9 new documentation pages
- 7 new API endpoints:
  - `POST /v1/stablefx/quotes` - Quote creation
  - `POST /v1/stablefx/trades` - Trade initiation
  - `GET /v1/stablefx/trades` - Trade listing
  - `GET /v1/stablefx/trades/{tradeId}` - Trade retrieval
  - `POST /v1/stablefx/trades/{tradeId}/signature-data` - Signature data
  - `POST /v1/stablefx/trades/{tradeId}/signatures` - Signature registration
  - `POST /v1/stablefx/trades/{tradeId}/fund` - Trade funding

---

# 4. Compliance Engine

## 4.1 Overview

**Compliance Engine** automates transaction screening for blockchain workflows, addressing AML, CTF, and jurisdictional requirements.

### Key Features
- Real-time transaction screening (flag high-risk before execution)
- Custom rules (adjustable by jurisdiction/industry)
- Address controls (allowlists/blocklists)
- Alert management and investigation tools
- Detailed audit trails and compliance data export

### Availability
Both testnet and mainnet, but restricted to **eligible customers only** (requires access request).

---

## 4.2 Transaction Screening

### Activation Modes
1. **Embedded**: Screening occurs implicitly during programmable wallet transactions
2. **Standalone**: Triggered explicitly through Compliance API

### Screening Endpoint
```
POST /v1/w3s/compliance/screening/addresses
Body: {
  "idempotencyKey": "UUID",
  "address": "0x...",
  "chain": "ETH-SEPOLIA"
}
```

### Response Structure
```json
{
  "result": "DENIED",
  "decision": {
    "ruleName": "Severe Sanctions Risk",
    "actions": ["DENY", "REVIEW", "FREEZE_WALLET"],
    "reasons": [{
      "source": "...",
      "riskScore": "Severe",
      "riskCategories": ["SANCTIONS"],
      "type": "Ownership"
    }]
  },
  "screeningDate": "ISO 8601"
}
```

### Recommended Actions
- **DENY**: Block transaction
- **REVIEW**: Flag for manual review
- **FREEZE_WALLET**: Suspend all wallet activity

### Embedded Response
Transaction retrieval includes `transactionScreeningEvaluation` field with matched rule, actions, and risk reasons.

---

## 4.3 Screening Quickstart

### Prerequisites
- Circle Developer Account with Compliance Engine access
- Node.js + Axios or cURL/Postman

### Implementation
```javascript
const response = await axios.post(
  'https://api.circle.com/v1/w3s/compliance/screening/addresses',
  { idempotencyKey: uuid(), address: '0x...', chain: 'ETH-SEPOLIA' },
  { headers: { Authorization: `Bearer ${apiKey}` } }
);
// Parse response.data.result, decision.ruleName, decision.actions
```

Applications parse `riskCategories`, `ruleName`, `type` to implement conditional compliance workflows.

---

## 4.4 Rule Management

### Rule Types
1. **Restrictive**: Block transactions or freeze wallets
2. **Alert-only**: Generate notifications without blocking

### Rule Components
- Name (auto-generated), Description, Actions, Criteria

### Rule Criteria Dimensions
- **Risk Category**: SANCTIONS, TERRORIST_FINANCING, CSAM, PEP, GAMBLING, HIGH_RISK_INDUSTRY, ILLICIT_BEHAVIOR, OTHER
- **Risk Score**: Severe, High, Medium, Low
- **Risk Type**: Ownership, Counterparty, Indirect

### Default Mandatory Rules
1. Circle's Sanctions Blocklist (OFAC)
2. Your blocklist (custom addresses)
3. Frozen (wallet freeze enforcement)
4. Your allowlist (whitelisted addresses)

### Watchlists
- **Blocklist**: Deny transactions from specified addresses
- **Allowlist**: Permit transactions from specified addresses
- Manage via Circle Console > Compliance Engine > Watchlists

---

## 4.5 Alert Management

### Dashboard Features
- Historical alert viewing with tabular interface
- Detailed alert-level information and entity data
- Wallet risk information linked to alerts

### Actions
- **Freeze Wallet**: Button in Entities > Action column
- **Add to Blocklist**: Direct from alert view
- **Close Alert**: Status dropdown in table header

---

## 4.6 Testing with Magic Values

Nine pre-configured rules on testnet. Use addresses with specific numeric suffixes:

| Rule | Suffix | Risk Score | Actions |
|------|--------|-----------|---------|
| Circle's Sanctions Blocklist | 9999 | Blocklist | DENY, REVIEW, FREEZE_WALLET |
| Frozen User Wallet | 8888 | Blocklist | DENY, REVIEW |
| Your Blocklist | 7777 | Blocklist | DENY, REVIEW |
| Severe Sanctions (Owner) | 8999 | Severe | DENY, REVIEW, FREEZE_WALLET |
| Severe Terrorist Financing (Owner) | 8899 | Severe | DENY, REVIEW, FREEZE_WALLET |
| Severe CSAM (Owner) | 8889 | Severe | DENY, REVIEW, FREEZE_WALLET |
| Severe Illicit Behavior (Owner) | 7779 | Severe | DENY, REVIEW, FREEZE_WALLET |
| High Illicit Behavior (Owner) | 7666 | High | REVIEW |
| High Gambling (Owner) | 7766 | High | REVIEW |

Generate vanity addresses using tools like vanity-eth.tk, import to MetaMask, fund via testnet faucets.

---

## 4.7 OFAC & W3S Compliance Requirements

### OFAC Sanctions
- Wallets must not transact with OFAC-sanctioned addresses
- Non-compliant wallets get temporary outbound restrictions
- Flagged funds require coordination with Circle support for quarantine
- Violations may result in account termination and regulatory reporting

### SDN List Reference
`https://www.treasury.gov/ofac/downloads/sdnlist.pdf`

---

# 5. Paymaster

## 5.1 Overview

**Circle Paymaster** is a network of permissionless token paymasters enabling end-users to pay gas fees in USDC instead of chain native tokens.

### Key Features
- **ERC-4337** v0.7 and v0.8 compliant
- **Permissionless**: No signup, API keys, or Circle account required
- **Onchain smart contract**: Works with any ERC-4337 wallet
- **10% surcharge** on gas costs (Arbitrum + Base only)
- Exclusively supports USDC

### Network Support

| Version | Networks |
|---------|----------|
| v0.7 | Arbitrum, Base |
| v0.8 | Arbitrum, Avalanche, Base, Ethereum, Optimism, Polygon, Unichain |

### How It Works
- Users can deploy smart contract accounts without native tokens
- Uses ERC-20 approvals and signed permits
- Circle maintains native gas token reserves across chains
- Automated swap and balance management

### vs Gas Station
| | Paymaster | Gas Station |
|--|-----------|-------------|
| Target | End-users | Developers |
| Fee | 10% | 5% |
| Payment | USDC by user | Sponsored by developer |

---

## 5.2 Addresses & Events

### Paymaster v0.7 Addresses
- **Testnet** (Arbitrum Sepolia, Arc Testnet, Base Sepolia): `0x31BE08D380A21fc740883c0BC434FcFc88740b58`

### Paymaster v0.8 Addresses
- **Mainnet** (Arbitrum, Avalanche, Base, Ethereum, Optimism, Polygon, Unichain): `0x0578cFB241215b77442a541325d6A4E6dFE700Ec`
- **Testnet** (8 networks including Ethereum Sepolia, Avalanche Fuji, Polygon Amoy): `0x3BA9A96eE3eFf3A69E2B18886AcF52027EFF8966`

### Contract Functions
1. **`_validatePaymasterUserOp`**: Validates user operations, charges prefund tokens before execution
2. **`_postOp`**: Refunds tokens when actual costs are known

### Events
**`UserOperationSponsored`**:
| Parameter | Type | Purpose |
|-----------|------|---------|
| `token` | IERC20 | ERC-20 token used |
| `sender` | address | Transaction originator |
| `userOpHash` | bytes32 | Operation identifier |
| `nativeTokenPrice` | uint256 | 1 ether = 1e18 wei in token |
| `actualTokenNeeded` | uint256 | Final cost |
| `feeTokenAmount` | uint256 | Slippage buffer |

---

## 5.3 Pay Gas Fees in USDC Quickstart

### Architecture
- v0.7: `toCircleSmartAccount` (traditional account-abstraction)
- v0.8: `toSimple7702SmartAccount` (EIP-7702 authorization-based)

### Prerequisites
```
OWNER_PRIVATE_KEY
RECIPIENT_ADDRESS
PAYMASTER_V07/V08_ADDRESS
USDC_ADDRESS
```

Dependencies: `viem`, `@circle-fin/modular-wallets-core`, `dotenv`

### Part 1: Account Initialization
```javascript
import { createPublicClient, http, getContract } from "viem";
import { arbitrumSepolia } from "viem/chains";
import { privateKeyToAccount } from "viem/accounts";

const owner = privateKeyToAccount(OWNER_PRIVATE_KEY);
const client = createPublicClient({ chain: arbitrumSepolia, transport: http() });
// Initialize smart account, verify USDC balance >= 1,000,000 units
```

### Part 2: Paymaster Configuration

**EIP-2612 Permit Generation:**
```javascript
const permitData = {
  types: { Permit: [...] },
  domain: { name: "USDC", version: "2", chainId: chainId, verifyingContract: usdcAddress },
  message: { owner: account.address, spender: paymasterAddress, value: permitAmount, nonce: ownerNonce, deadline: maxUint256 }
};
const signature = await account.signTypedData(permitData);
```

**Encode Paymaster Data:**
```javascript
const paymasterData = encodePacked(
  ["uint8", "address", "uint256", "bytes"],
  [1, usdcAddress, permitAmount, signature]
);
```

**Paymaster Object:**
```javascript
const paymaster = {
  getPaymasterData: async () => ({
    paymaster: paymasterAddress,
    paymasterData: paymasterData,
    paymasterVerificationGasLimit: 200000n,
    paymasterPostOpGasLimit: 15000n,
    isFinal: true
  })
};
```

### Part 3: Submit User Operation
```javascript
const bundlerClient = createBundlerClient({
  chain: arbitrumSepolia,
  transport: http(pimlicoRpcUrl),
  account: account,
  paymaster: paymaster,
  userOperation: {
    estimateFeesPerGas: async () => {
      const fees = await bundlerClient.request({ method: "pimlico_getUserOperationGasPrice" });
      return { maxFeePerGas: hexToBigInt(fees.fast.maxFeePerGas), maxPriorityFeePerGas: hexToBigInt(fees.fast.maxPriorityFeePerGas) };
    }
  }
});

// For v0.8: Sign EIP-7702 authorization first
const authorization = await owner.signAuthorization({
  chainId: chain.id,
  nonce: await client.getTransactionCount({ address: owner.address }),
  contractAddress: account.authorization.address
});

const hash = await bundlerClient.sendUserOperation({
  calls: [{ to: recipientAddress, data: transferCallData }],
  authorization // v0.8 only
});

const receipt = await bundlerClient.waitForUserOperationReceipt({ hash });
```

### Debugging
- User operation hash differs from transaction hash
- Use txHash in Arbiscan, userOpHash in JiffyScan

---

# Summary of All Services

| Service | Purpose | Settlement | Key Technology |
|---------|---------|-----------|---------------|
| **Circle Mint** | Institutional USDC/EURC access & redemption | Wire/blockchain | REST API + webhooks |
| **CPN** | Cross-border USDC->fiat payments | Blockchain + fiat rails | JWE encryption, EIP-712, Permit2 |
| **StableFX** | Institutional stablecoin FX (USDC/EURC) | Arc blockchain escrow | RFQ + FxEscrow contract + Permit2 |
| **Compliance Engine** | Transaction screening (AML/CTF) | N/A | Rule engine + sanctions lists |
| **Paymaster** | Gas fees in USDC | ERC-4337 bundler | Smart contract paymaster + EIP-2612 permits |
