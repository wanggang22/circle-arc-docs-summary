# Circle W3S API Reference - Comprehensive Summary

> Covers 111 API endpoints across Contracts and Wallets services.
> Base URL: `https://api.circle.com`
> Auth: Bearer token format `PREFIX:ID:SECRET` (unless noted otherwise)

---

## Table of Contents

1. [Common / Shared APIs](#1-common--shared-apis)
2. [Smart Contract Platform](#2-smart-contract-platform)
3. [BUIDL (Read-Only Wallet Queries)](#3-buidl-read-only-wallet-queries)
4. [Compliance](#4-compliance)
5. [Programmable Wallets (Config / Tokens)](#5-programmable-wallets-config--tokens)
6. [Developer-Controlled Wallets](#6-developer-controlled-wallets)
7. [User-Controlled Wallets](#7-user-controlled-wallets)
8. [Cross-Cutting Concepts](#8-cross-cutting-concepts)

---

## 1. Common / Shared APIs

These endpoints are identical for both the Contracts API and the Wallets API.

### 1.1 Health Check

| Detail | Value |
|--------|-------|
| Method | `GET /ping` |
| Auth | None required |
| Response | `{ "message": "pong" }` |
| Notes | Simple service availability check. No parameters needed. |

### 1.2 Notification Subscriptions (Webhooks)

#### Create Subscription
| Detail | Value |
|--------|-------|
| Method | `POST /v2/notifications/subscriptions` |
| Body | `endpoint` (required, HTTPS URL), `notificationTypes` (optional array, supports wildcards `*`) |
| Response 200 | `{ data: { id, name, endpoint, enabled, notificationTypes, restricted, createDate, updateDate } }` |
| Notes | Endpoint must be publicly accessible HTTPS, respond with 2XX. Omitting `notificationTypes` creates unrestricted webhook (all events). |

**Notification type values:** `*`, `transactions.*`, `transactions.inbound`, `transactions.outbound`, `challenges.*`, `contracts.*`, `modularWallet.*`, `travelRule.*`, `rampSession.*`, plus specific sub-types.

#### Get All Subscriptions
| Detail | Value |
|--------|-------|
| Method | `GET /v2/notifications/subscriptions` |
| Parameters | None |
| Response 200 | `{ data: [ ...Subscription ] }` |

#### Get Single Subscription
| Detail | Value |
|--------|-------|
| Method | `GET /v2/notifications/subscriptions/{id}` |
| Path Params | `id` (UUID, required) |
| Response 200 | `{ data: Subscription }` |
| Errors | 401, 404 |

#### Update Subscription
| Detail | Value |
|--------|-------|
| Method | `PATCH /v2/notifications/subscriptions/{id}` |
| Path Params | `id` (UUID, required) |
| Body | `name` (string, required), `enabled` (boolean, required) |
| Response 200 | Updated Subscription object |
| Errors | 400, 401, 404 |

#### Delete Subscription
| Detail | Value |
|--------|-------|
| Method | `DELETE /v2/notifications/subscriptions/{id}` |
| Path Params | `id` (UUID, required) |
| Response 204 | Empty body |
| Errors | 401, 404 |

#### Get Notification Signature Public Key
| Detail | Value |
|--------|-------|
| Method | `GET /v2/notifications/publicKey/{id}` |
| Path Params | `id` (UUID, from `X-Circle-Key-Id` webhook header) |
| Response 200 | `{ data: { id, algorithm, publicKey, createDate } }` |
| Notes | Use to verify `X-Circle-Signature` header on incoming webhooks. Algorithm typically `ECDSA_SHA_256`. |

---

## 2. Smart Contract Platform

Base path: `/v1/w3s/contracts`

### Supported Blockchains (all contract endpoints)
ETH, ETH-SEPOLIA, MATIC, MATIC-AMOY, ARB, ARB-SEPOLIA, UNI, UNI-SEPOLIA, BASE, BASE-SEPOLIA, OP, OP-SEPOLIA, AVAX, AVAX-FUJI, ARC-TESTNET, MONAD, MONAD-TESTNET

### 2.1 Contract Deployment

#### Deploy Contract (Custom Bytecode)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/contracts/deploy` |
| Required Body | `idempotencyKey` (UUID), `name` (alphanumeric), `walletId` (UUID), `abiJson` (stringified JSON), `bytecode`, `entitySecretCiphertext` (base64), `blockchain` (enum) |
| Optional Body | `description`, `constructorParameters` (array), `feeLevel` (LOW/MEDIUM/HIGH), `gasLimit`, `gasPrice`, `maxFee`, `priorityFee`, `refId` |
| Response 201/200 | `{ data: { contractId, transactionId } }` |
| Key Rules | `feeLevel` and manual gas params are mutually exclusive. Entity secret ciphertext must be unique per request. |

#### Deploy Contract from Template
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/templates/{id}/deploy` |
| Path Params | `id` (template UUID) |
| Required Body | `idempotencyKey`, `blockchain`, `walletId`, `name`, `entitySecretCiphertext` |
| Optional Body | `description`, `templateParameters` (JSON object), `feeLevel`, `gasLimit`, `gasPrice`, `maxFee`, `priorityFee`, `refId` |
| Response 201/200 | `{ data: { contractIds: [UUID], transactionId } }` |

#### Estimate Contract Deployment Fee
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/contracts/deploy/estimateFee` |
| Required Body | `bytecode` |
| Optional Body | `walletId` OR (`sourceAddress` + `blockchain`), `abiJson`, `constructorSignature`, `constructorParameters` |
| Response 200 | `{ data: { high: FeeObj, medium: FeeObj, low: FeeObj } }` |

**FeeObj:** `{ gasLimit, gasPrice, maxFee, priorityFee, baseFee, networkFee, networkFeeRaw, l1Fee }`

#### Estimate Template Deployment Fee
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/templates/{id}/deploy/estimateFee` |
| Required | `blockchain`, plus `walletId` OR `sourceAddress` |
| Optional | `templateParameters` |
| Response 200 | Same 3-tier fee structure as above |

### 2.2 Contract Management

#### Import Contract
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/contracts/import` |
| Required Body | `idempotencyKey`, `name`, `address`, `blockchain` |
| Optional Body | `description`, `refreshVerification` (boolean) |
| Response 201/200 | `{ data: { contract: ContractObj } }` |
| Notes | Pulls ABI/source from blockchain explorers for verified contracts. `contractInputType` = IMPORT. |

#### List Contracts
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/contracts` |
| Query Params | `blockchain`, `contractInputType` (IMPORT/BYTECODE/TEMPLATE/AUTO_IMPORT), `deployerAddress`, `name`, `status` (PENDING/FAILED/COMPLETE), `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` (default 10, max 50) |
| Response 200 | `{ data: { contracts: [ContractObj] } }` |

#### Get Contract
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/contracts/{id}` |
| Path Params | `id` (UUID, Circle internal ID, NOT on-chain address) |
| Response 200 | `{ data: { contract: ContractObj } }` |

**ContractObj fields:** id, name, contractInputType, createDate, updateDate, archived, blockchain, status, verificationStatus (VERIFIED/UNVERIFIED), contractAddress, deployerAddress, txHash, functions, events, sourceCode, abiJson

#### Update Contract
| Detail | Value |
|--------|-------|
| Method | `PATCH /v1/w3s/contracts/{id}` |
| Optional Body | `name`, `description`, `archived` (boolean) |
| Response 200 | Updated ContractObj |

#### Query Contract (Read-Only State)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/contracts/query` |
| Required Body | `address`, `blockchain` |
| Interaction (choose one) | `abiFunctionSignature` + `abiParameters` OR `callData` (hex) |
| Optional Body | `abiJson`, `fromAddress` |
| Response 200 | `{ data: { outputValues: [], outputData: "0x..." } }` |
| Notes | Read-only operation for contract state inspection. |

### 2.3 Event Monitors

#### Create Event Monitor
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/contracts/monitors` |
| Required Body | `blockchain`, `contractAddress`, `eventSignature` (no spaces), `idempotencyKey` |
| Response 200/201 | `{ data: { eventMonitor: { id, blockchain, contractAddress, eventSignature, eventSignatureHash, isEnabled, createDate, updateDate } } }` |

#### List Event Monitors
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/contracts/monitors` |
| Query Params | `contractAddress`, `blockchain`, `eventSignature`, `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` (default 10, max 50) |
| Response 200 | `{ data: { eventMonitors: [...] } }` |

#### Update Event Monitor
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/contracts/monitors/{id}` |
| Body | `isEnabled` (boolean, required) |
| Response 200 | Updated eventMonitor object |

#### Delete Event Monitor
| Detail | Value |
|--------|-------|
| Method | `DELETE /v1/w3s/contracts/monitors/{id}` |
| Response 200 | Empty body |

#### List Event Logs
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/contracts/events` |
| Query Params | `contractAddress`, `blockchain`, `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` |
| Response 200 | `{ data: { eventLogs: [{ id, blockHash, blockHeight, blockchain, contractAddress, data, eventSignature, eventSignatureHash, logIndex, topics, txHash, userOpHash, firstConfirmDate }] } }` |

---

## 3. BUIDL (Read-Only Wallet Queries)

Base path: `/v1/w3s/buidl` -- lightweight read-only endpoints for querying wallet data.

### 3.1 Transfers

#### Get Transfer
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/transfers/{id}` |
| Response 200 | `{ data: { transfer: { id, walletId, amount, blockchain, from, to, state, tokenId, transferType, txHash, walletAddress, ... } } }` |
| States | CONFIRMED, COMPLETE, FAILED |
| Transfer Types | INBOUND_TRANSFER, OUTBOUND_TRANSFER |

#### List Transfers
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/transfers` |
| Required Params | `walletAddresses` (comma-separated) |
| Optional Params | `blockchain`, `state`, `transferType`, `txHash`, `userOpHash`, `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` |
| Response 200 | `{ data: { transfers: [...] } }` |

### 3.2 User Operations

#### Get User Operation
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/userOps/{id}` |
| Response 200 | `{ data: { userOperation: { id, blockchain, state, userOpHash, txHash, actualGasUsed, actualGasCost, blockHeight, ... } } }` |
| States | SENT, CONFIRMED, COMPLETE, FAILED |

#### List User Operations
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/userOps` |
| Optional Params | `blockchain`, `refId`, `senders`, `state`, `txHash`, `userOpHash`, `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` |
| Response 200 | `{ data: { userOperations: [...] } }` |

### 3.3 Wallet Balances & NFTs

#### Get Balances by Blockchain Address
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/wallets/{blockchain}/{address}/balances` |
| Optional Query | `standard` (ERC20), `name`, `tokenAddress`, pagination params |
| Response 200 | `{ data: { tokenBalances: [{ amount, token: { id, name, symbol, decimals, standard, blockchain, isNative, tokenAddress }, updateDate }] } }` |

#### Get Balances by Wallet ID
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/wallets/{id}/balances` |
| Optional Query | Same as above |
| Response 200 | Same tokenBalances structure |

#### Get NFTs by Blockchain Address
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/wallets/{blockchain}/{address}/nfts` |
| Optional Query | `standard` (ERC721/ERC1155), `name`, `tokenAddress`, pagination params |
| Response 200 | `{ data: { nfts: [{ nftTokenId, metadata, amount, token: {...}, updateDate }] } }` |

#### Get NFTs by Wallet ID
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/buidl/wallets/{id}/nfts` |
| Optional Query | Same as above |
| Response 200 | Same NFT structure |

---

## 4. Compliance

#### Screen Address
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/compliance/screening/addresses` |
| Required Body | `idempotencyKey`, `address`, `chain` (ETH, MATIC, SOL, BTC, etc.) |
| Response 200 | `{ result: "APPROVED"/"DENIED", decision: { ruleName, actions, screeningDate, riskSignals }, id, address, chain, details, alertId }` |
| Risk Scores | UNKNOWN, LOW, MEDIUM, HIGH, SEVERE, BLOCKLIST |
| Risk Categories | SANCTIONS, GAMBLING, ILLICIT_BEHAVIOR, etc. |
| Risk Types | OWNERSHIP, COUNTERPARTY, INDIRECT |

---

## 5. Programmable Wallets (Config / Tokens)

### 5.1 Entity Configuration

#### Get Entity Config
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/config/entity` |
| Response 200 | `{ data: { appId: UUID } }` |

#### Get Entity Public Key
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/config/entity/publicKey` |
| Response 200 | `{ data: { publicKey: string } }` |
| Notes | Used to encrypt entity secret for API requests. |

### 5.2 Monitored Tokens

Monitored tokens control which tokens appear by default in wallet balance queries.

#### Add Monitored Tokens
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/config/entity/monitoredTokens` |
| Body | `tokenIds` (array of UUIDs, min 1) |
| Response 201/200 | `{ data: { scope: "SELECTED"/"MONITOR_ALL", tokens: [Token] } }` |

#### Update (Upsert) Monitored Tokens
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/config/entity/monitoredTokens` |
| Body | `tokenIds` (array of UUIDs) |
| Response 200 | Same structure as above |

#### Delete Monitored Tokens
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/config/entity/monitoredTokens/delete` |
| Body | `tokenIds` (array of UUIDs, min 1) |
| Response 200 | Empty body |

#### List Monitored Tokens
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/config/entity/monitoredTokens` |
| Optional Query | `blockchain`, `tokenAddress`, `symbol`, `from`, `to`, pagination params |
| Response 200 | `{ data: { scope, tokens: [...] } }` |

#### Update Monitored Tokens Scope
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/config/entity/monitoredTokens/scope` |
| Body | `scope` ("SELECTED" or "MONITOR_ALL") |
| Response 200 | Empty body |

### 5.3 Faucet (Testnet Tokens)

#### Request Testnet Tokens
| Detail | Value |
|--------|-------|
| Method | `POST /v1/faucet/drips` |
| Body | `address` (required), `blockchain` (required, testnet only), `native` (bool), `usdc` (bool), `eurc` (bool) |
| Response 204 | Empty body |
| Supported Chains | ETH-SEPOLIA, AVAX-FUJI, MATIC-AMOY, SOL-DEVNET, ARB-SEPOLIA, UNI-SEPOLIA, BASE-SEPOLIA, OP-SEPOLIA, APTOS-TESTNET, ARC-TESTNET, MONAD-TESTNET |

---

## 6. Developer-Controlled Wallets

### 6.1 Wallet Sets

#### Create Wallet Set
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/walletSets` |
| Required Body | `entitySecretCiphertext`, `idempotencyKey` |
| Optional Body | `name` |
| Response 201/200 | `{ data: { walletSet: { id, custodyType, createDate, updateDate } } }` |
| Limits | Up to 1,000 wallet sets per account, each supporting up to 10 million wallets |

#### Get Wallet Set
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/walletSets/{id}` |
| Response 200 | WalletSet object |

#### List Wallet Sets
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/walletSets` |
| Query Params | `from`, `to`, `pageAfter`, `pageBefore`, `pageSize` (max 50), `order` (ASC/DESC) |
| Response 200 | Array of WalletSet objects |

#### Update Wallet Set
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/developer/walletSets/{id}` |
| Body | `name` (string) |
| Response 200 | Updated WalletSet |

### 6.2 Wallets

#### Create Wallets
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/wallets` |
| Required Body | `blockchains` (array), `walletSetId`, `idempotencyKey`, `entitySecretCiphertext` |
| Optional Body | `accountType` (EOA/SCA, default EOA), `count` (1-200, default 1), `metadata` (array of {name, refId}) |
| Response 201/200 | `{ data: { wallets: [WalletObj] } }` |
| Notes | SOL and APTOS do not support SCA. |

**WalletObj fields:** id, address, blockchain, accountType, state (LIVE/FROZEN), custodyType (DEVELOPER/ENDUSER), walletSetId, name, refId, scaCore, createDate, updateDate

#### Derive Wallet (by wallet ID + target blockchain)
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/developer/wallets/{id}/blockchains/{blockchain}` |
| Optional Body | `metadata` (name, refId) |
| Response 201/200 | Single WalletObj |
| Notes | Creates wallet on target chain with same address. EVM-only. |

#### Derive Wallet by Address
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/developer/wallets/derive` |
| Required Body | `sourceBlockchain`, `walletAddress`, `targetBlockchain` |
| Optional Body | `metadata` |
| Response 201/200 | Single WalletObj |

#### Get Wallet
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets/{id}` |
| Response 200 | `{ data: { wallet: WalletObj } }` |

#### List Wallets
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets` |
| Query Params | `address`, `blockchain`, `scaCore`, `walletSetId`, `refId`, `from`, `to`, pagination, `order` |
| Response 200 | `{ data: { wallets: [...] } }` |

#### List Wallets with Balances
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/developer/wallets/balances` |
| Required Query | `blockchain` |
| Optional Query | `address`, `walletSetId`, `refId`, `scaCore`, `amount__gte`, `tokenAddress`, `from`, `to`, pagination |
| Response 200 | Wallets array with embedded `tokenBalances` |

#### Update Wallet
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/wallets/{id}` |
| Body | `name` (optional), `refId` (optional) |
| Response 200 | Updated WalletObj |

#### List Wallet Balances
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets/{id}/balances` |
| Optional Query | `includeAll` (bool), `name`, `tokenAddress`, `standard`, pagination |
| Response 200 | `{ data: { tokenBalances: [...] } }` |
| Notes | Aptos excludes AIP-21 secondary storage tokens. |

#### List Wallet NFTs
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets/{id}/nfts` |
| Optional Query | `includeAll`, `name`, `tokenAddress`, `standard`, pagination |
| Response 200 | `{ data: { nfts: [...] } }` |

### 6.3 Transactions

#### Create Transfer Transaction
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/transactions/transfer` |
| Required Body | `idempotencyKey`, `destinationAddress`, `entitySecretCiphertext` |
| Wallet ID (one of) | `walletId` OR (`walletAddress` + `blockchain`) |
| Optional Body | `amounts`, `tokenId`, `tokenAddress`, `nftTokenIds`, `feeLevel`, `gasLimit`, `gasPrice`, `maxFee`, `priorityFee`, `refId` |
| Response 201/200 | `{ data: { id, state } }` |
| Transaction States | INITIATED, QUEUED, SENT, CONFIRMED, COMPLETE, FAILED, DENIED, CANCELLED, CLEARED, STUCK |
| Notes | ERC-721 requires `amounts: ["1"]`. Fee level and manual gas params are mutually exclusive. |

#### Create Contract Execution Transaction
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/transactions/contractExecution` |
| Required Body | `idempotencyKey`, `contractAddress`, `entitySecretCiphertext` |
| Interaction (one of) | `abiFunctionSignature` + `abiParameters` OR `callData` (hex) |
| Optional Body | `amount` (for payable), wallet ID params, fee params, `refId` |
| Response 201/200 | `{ data: { id, state } }` |

#### Create Wallet Upgrade Transaction
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/transactions/walletUpgrade` |
| Required Body | `idempotencyKey`, `walletId`, `entitySecretCiphertext`, `newScaCore` (currently `circle_6900_singleowner_v2`) |
| Optional Body | Fee params, `refId` |
| Response 201/200 | `{ data: { id, state } }` |

#### Accelerate Transaction
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/transactions/{id}/accelerate` |
| Required Body | `idempotencyKey`, `entitySecretCiphertext` |
| Response 200 | `{ data: { id } }` |
| Notes | Additional gas fees may apply. |

#### Cancel Transaction
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/transactions/{id}/cancel` |
| Required Body | `idempotencyKey`, `entitySecretCiphertext` |
| Response 200 | `{ data: { id, state } }` |
| Notes | No guarantee if already confirmed on-chain. Gas fees may still apply. |

#### Get Transaction
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/transactions/{id}` |
| Optional Query | `txType` (INBOUND/OUTBOUND) |
| Response 200 | `{ data: { transaction: TransactionObj } }` |
| Notes | Compliance Engine customers get `transactionScreeningEvaluation` field. |

#### List Transactions
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/transactions` |
| Query Params | `blockchain`, `custodyType`, `destinationAddress`, `operation` (TRANSFER/CONTRACT_EXECUTION/CONTRACT_DEPLOYMENT), `state`, `txHash`, `txType`, `walletIds`, `from`, `to`, pagination, `order`, `includeAll` |
| Response 200 | `{ data: { transactions: [...] } }` |

#### Get Lowest Nonce Pending Transaction
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/transactions/lowestNonceTransaction` |
| Required Query | `blockchain` |
| Optional Query | `address`, `walletId` (at least one identifier needed) |
| Response 200 | `{ data: { transaction, feeInfo: { newHighEstimatedFee, feeDifferenceAmount } } }` |
| Response 204 | No pending transaction found |

### 6.4 Fee Estimation

#### Estimate Contract Execution Fee
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/transactions/contractExecution/estimateFee` |
| Required Body | `contractAddress` |
| Optional Body | `abiFunctionSignature`/`abiParameters` OR `callData`, `amount`, `blockchain`, `sourceAddress` OR `walletId` |
| Response 200 | 3-tier fee structure (high/medium/low) + optional ERC-4337 gas fields |

#### Estimate Transfer Fee
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/transactions/transfer/estimateFee` |
| Required Body | `amounts`, `destinationAddress` |
| Optional Body | `blockchain`, `tokenId`, `tokenAddress`, `walletId` OR `sourceAddress`, `nftTokenIds` |
| Response 200 | Same 3-tier fee structure |

**Fee fields per tier:** gasLimit, gasPrice (non-EIP-1559), maxFee + priorityFee (EIP-1559), baseFee, networkFee, networkFeeRaw, l1Fee (L2 chains)
**ERC-4337 fields (SCA only):** callGasLimit, verificationGasLimit, preVerificationGas

#### Get Fee Parameters
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/developer/transactions/feeParameters` |
| Required Query | `blockchain` |
| Optional Query | `accountType` (SCA/EOA, default EOA) |
| Response 200 | `{ data: { high: FeeParams, medium: FeeParams, low: FeeParams } }` |

### 6.5 Token Lookup

#### Get Token by ID
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/tokens/{id}` |
| Response 200 | `{ data: { token: { id, name, symbol, standard, blockchain, decimals, isNative, tokenAddress, createDate, updateDate } } }` |

### 6.6 Address Validation

#### Validate Address
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/transactions/validateAddress` |
| Body | `address` (required), `blockchain` (required) |
| Response 200 | `{ data: { isValid: boolean } }` |

### 6.7 Signing

#### Sign Message
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/sign/message` |
| Required Body | `message`, `entitySecretCiphertext` |
| Wallet (one of) | `walletId` OR (`walletAddress` + `blockchain`) |
| Optional Body | `encodedByHex` (bool), `memo` |
| Response 200 | `{ data: { signature } }` |
| Notes | EIP-191 for Ethereum, Ed25519 for Solana/Aptos. |

#### Sign Typed Data (EIP-712)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/sign/typedData` |
| Required Body | `data` (EIP-712 typed data), `entitySecretCiphertext` |
| Wallet (one of) | `walletId` OR (`walletAddress` + `blockchain`) |
| Optional Body | `memo` |
| Response 200 | `{ data: { signature } }` |

#### Sign Transaction
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/sign/transaction` |
| Required Body | `entitySecretCiphertext` |
| Wallet (one of) | `walletId` OR (`walletAddress` + `blockchain`) |
| Transaction (one of) | `rawTransaction` (base64/hex) OR `transaction` (JSON, EVM-only) |
| Optional Body | `memo` |
| Response 200 | `{ data: { signature, signedTransaction, txHash } }` |
| Supported Chains | SOL, SOL-DEVNET, NEAR, NEAR-TESTNET, EVM, EVM-TESTNET |

#### Sign Delegate Action (NEAR only)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/developer/sign/delegateAction` |
| Required Body | `walletId`, `unsignedDelegateAction` (base64), `entitySecretCiphertext` |
| Response 200 | `{ data: { signature, signedDelegateAction } }` |
| Supported Chains | NEAR, NEAR-TESTNET only |

---

## 7. User-Controlled Wallets

User-controlled wallets use a challenge-based flow. Most mutating operations return a `challengeId` that must be completed via the client SDK with user PIN.

### 7.1 User Management

#### Create User
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/users` |
| Body | `userId` (string, 5-50 chars, required) |
| Response 201 | `{ data: { id, createDate, pinStatus, status, securityQuestionStatus, pinDetails, securityQuestionDetails } }` |
| Pin Statuses | ENABLED, UNSET, LOCKED |
| User Statuses | ENABLED, DISABLED |

#### Get User by ID
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/users/{id}` |
| Response 200 | User object with pin/security details |

#### Get User by Token
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/user` |
| Required Header | `X-User-Token` (JWT) |
| Response 200 | User object |

#### List Users
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/users` |
| Query Params | `pinStatus`, `securityQuestionStatus`, `from`, `to`, pagination, `order` |
| Response 200 | `{ data: { users: [...] } }` |

### 7.2 Authentication / Tokens

#### Create User Token (Session)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/users/token` |
| Body | `userId` (required, 5-50 chars) |
| Response 200 | `{ data: { userToken: JWT (60min expiry), encryptionKey } }` |

#### Refresh User Token
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/users/token/refresh` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey`, `refreshToken`, `deviceId` |
| Response 200 | `{ data: { userToken, encryptionKey, userID, refreshToken } }` |

#### Get Device Token for Email OTP Login
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/users/email/token` |
| Body | `idempotencyKey`, `deviceId`, `email` |
| Response 200 | `{ data: { deviceToken (JWT), deviceEncryptionKey (base64), otpToken (JWT) } }` |

#### Get Device Token for Social Login
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/users/social/token` |
| Body | `idempotencyKey`, `deviceId` |
| Response 200 | `{ data: { deviceToken (JWT), deviceEncryptionKey (base64) } }` |

#### Resend OTP Email
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/users/email/resendOTP` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey`, `otpToken`, `email`, `deviceId` |
| Response 200 | `{ data: { otpToken } }` |
| Notes | Prior OTP expires when new one is sent. |

### 7.3 PIN Challenges

#### Create PIN Setup Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/pin` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey` |
| Response 201/200 | `{ data: { challengeId } }` |
| Notes | Does NOT create wallets. PIN setup only. |

#### Create PIN Update Challenge
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/user/pin` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey` |
| Response 201/200 | `{ data: { challengeId } }` |

#### Create PIN Restore Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/pin/restore` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey` |
| Response 201/200 | `{ data: { challengeId } }` |
| Notes | Uses security question verification. |

### 7.4 User Initialization & Wallets

#### Initialize User (Create User + PIN + Wallets)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/initialize` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey` (required), `accountType` (optional, SCA/EOA), `blockchains` (optional array), `metadata` (optional) |
| Response 201/200 | `{ data: { challengeId } }` |

#### Create User Wallets
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/wallets` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey`, `blockchains` (required), `accountType` (optional), `metadata` (optional) |
| Response 201/200 | `{ data: { challengeId } }` |

#### Get Wallet
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets/{id}` |
| Required Headers | `X-User-Token` |
| Response 200 | WalletObj |

#### List Wallets
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets` |
| Required Headers | `X-User-Token` |
| Query Params | `address`, `blockchain`, `scaCore`, `walletSetId`, `refId`, `from`, `to`, pagination, `order` |
| Response 200 | `{ data: { wallets: [...] } }` |

#### Update Wallet
| Detail | Value |
|--------|-------|
| Method | `PUT /v1/w3s/wallets/{id}` |
| Required Headers | `X-User-Token` |
| Body | `name`, `refId` (both optional) |
| Response 200 | Updated WalletObj |

#### List Wallet Balances
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets/{id}/balances` |
| Required Headers | `X-User-Token` |
| Optional Query | `includeAll`, `name`, `tokenAddress`, `standard`, pagination |
| Response 200 | `{ data: { tokenBalances: [...] } }` |

#### List Wallet NFTs
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/wallets/{id}/nfts` |
| Required Headers | `X-User-Token` |
| Optional Query | Same as balances |
| Response 200 | `{ data: { nfts: [...] } }` |

### 7.5 Transaction Challenges

All transaction operations return a `challengeId` that must be completed by the user via SDK.

#### Create Transfer Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/transactions/transfer` |
| Required Headers | `X-User-Token` |
| Required Body | `walletId`, `destinationAddress`, `idempotencyKey` |
| Optional Body | `amounts`, `tokenId`, `tokenAddress`, `blockchain`, `feeLevel`, gas params, `nftTokenIds`, `refId` |
| Response 201/200 | `{ data: { challengeId } }` |

#### Create Contract Execution Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/transactions/contractExecution` |
| Required Headers | `X-User-Token` |
| Required Body | `walletId`, `contractAddress`, `idempotencyKey` |
| Interaction | `abiFunctionSignature` + `abiParameters` OR `callData` |
| Optional Body | `amount`, `refId`, fee params |
| Response 201/200 | `{ data: { challengeId } }` |

#### Create Wallet Upgrade Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/transactions/walletUpgrade` |
| Required Headers | `X-User-Token` |
| Required Body | `idempotencyKey`, `walletId`, `newScaCore` |
| Optional Body | Fee params, `refId` |
| Response 201/200 | `{ data: { challengeId } }` |

#### Create Accelerate Transaction Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/transactions/{id}/accelerate` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey` |
| Response 201/200 | `{ data: { challengeId } }` |

#### Create Cancel Transaction Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/transactions/{id}/cancel` |
| Required Headers | `X-User-Token` |
| Body | `idempotencyKey` |
| Response 201/200 | `{ data: { challengeId } }` |

### 7.6 Fee Estimation (User-Controlled)

Same endpoints as developer-controlled but require `X-User-Token` header:

| Endpoint | Method |
|----------|--------|
| Contract execution fee | `POST /v1/w3s/transactions/contractExecution/estimateFee` |
| Transfer fee | `POST /v1/w3s/transactions/transfer/estimateFee` |

### 7.7 Transaction & Token Queries

#### Get Transaction
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/transactions/{id}` |
| Required Headers | `X-User-Token` |
| Optional Query | `txType` (INBOUND/OUTBOUND) |
| Response 200 | TransactionObj |

#### List Transactions
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/transactions` |
| Required Headers | `X-User-Token` |
| Query Params | `blockchain`, `destinationAddress`, `operation`, `state`, `txHash`, `txType`, `userId`, `walletIds`, `from`, `to`, pagination, `order`, `includeAll` |
| Response 200 | `{ data: { transactions: [...] } }` |

#### Get Lowest Nonce Pending Transaction
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/transactions/lowestNonceTransaction` |
| Required Query | `blockchain`, plus `address` or `walletId` |
| Response 200/204 | Transaction with fee info, or empty |

#### Get Token by ID
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/tokens/{id}` |
| Response 200 | TokenObj |

#### Validate Address
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/transactions/validateAddress` |
| Body | `address`, `blockchain` |
| Response 200 | `{ data: { isValid } }` |

### 7.8 Challenge Management

#### Get Challenge
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/user/challenges/{id}` |
| Required Headers | `X-User-Token` |
| Response 200 | `{ data: { challenge: { id, type, status, correlationIds, errorCode, errorMessage } } }` |

**Challenge Types:** INITIALIZE, SET_PIN, CHANGE_PIN, SET_SECURITY_QUESTIONS, CREATE_WALLET, RESTORE_PIN, CREATE_TRANSACTION, ACCELERATE_TRANSACTION, CANCEL_TRANSACTION, CONTRACT_EXECUTION, WALLET_UPGRADE, SIGN_MESSAGE, SIGN_TYPEDDATA, SIGN_TRANSACTION

**Challenge Statuses:** PENDING, IN_PROGRESS, COMPLETE, FAILED, EXPIRED

#### List Challenges
| Detail | Value |
|--------|-------|
| Method | `GET /v1/w3s/user/challenges` |
| Required Headers | `X-User-Token` |
| Optional Query | `status` (PENDING/IN_PROGRESS) |
| Response 200 | `{ data: { challenges: [...] } }` |

### 7.9 Signing Challenges

#### Sign Message Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/sign/message` |
| Required Headers | `X-User-Token` |
| Body | `walletId`, `message` (required), `encodedByHex` (optional), `memo` (optional) |
| Response 201/200 | `{ data: { challengeId } }` |

#### Sign Typed Data Challenge (EIP-712)
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/sign/typedData` |
| Required Headers | `X-User-Token` |
| Body | `walletId`, `data` (EIP-712 typed data), `memo` (optional) |
| Response 201/200 | `{ data: { challengeId } }` |

#### Sign Transaction Challenge
| Detail | Value |
|--------|-------|
| Method | `POST /v1/w3s/user/sign/transaction` |
| Required Headers | `X-User-Token` |
| Body | `walletId`, `rawTransaction` OR `transaction` (EVM-only), `memo` (optional) |
| Response 201/200 | `{ data: { challengeId } }` |
| Supported Chains | SOL, SOL-DEVNET, EVM, EVM-TESTNET |

---

## 8. Cross-Cutting Concepts

### 8.1 Authentication

All authenticated endpoints use Bearer token: `Authorization: Bearer PREFIX:ID:SECRET`

User-controlled wallet endpoints additionally require: `X-User-Token: <JWT>`

### 8.2 Idempotency

All mutating (POST/PUT) endpoints accept `idempotencyKey` (UUID v4). Reusing the same key returns the original response, ensuring exactly-once execution. The `entitySecretCiphertext` must be unique per request.

### 8.3 Pagination

All list endpoints use cursor-based pagination:
- `pageAfter` (UUID): cursor for next page
- `pageBefore` (UUID): cursor for previous page (do NOT combine with pageAfter)
- `pageSize`: items per page (default 10, max 50)
- Response includes `Link` header with rel=self/first/next/prev URLs

### 8.4 Request Tracing

`X-Request-Id` header (UUID) can be provided by the developer or is auto-generated by Circle. Used for log tracing and support communication. Non-UUID values are accepted but ignored by logging systems.

### 8.5 Supported Blockchains (Full List)

**Mainnets:** ETH, MATIC, ARB, UNI, BASE, OP, AVAX, SOL, NEAR, APTOS, MONAD
**Testnets:** ETH-SEPOLIA, MATIC-AMOY, ARB-SEPOLIA, UNI-SEPOLIA, BASE-SEPOLIA, OP-SEPOLIA, AVAX-FUJI, SOL-DEVNET, NEAR-TESTNET, APTOS-TESTNET, ARC-TESTNET, MONAD-TESTNET
**Generic:** EVM, EVM-TESTNET

### 8.6 Account Types

- **EOA** (Externally Owned Account): Default, available on all chains
- **SCA** (Smart Contract Account): Available on EVM chains only (not SOL/APTOS)
  - SCA Core versions: `circle_4337_v1`, `circle_6900_singleowner_v1`, `circle_6900_singleowner_v2`, `circle_6900_singleowner_v3`

### 8.7 Transaction States

INITIATED -> QUEUED -> SENT -> CONFIRMED -> COMPLETE
Alternative outcomes: FAILED, DENIED, CANCELLED, CLEARED, STUCK

### 8.8 Error Response Format

All errors follow a consistent format:
```json
{
  "code": 400,
  "message": "Description of the error"
}
```

Common HTTP status codes: 400 (Bad Request), 401 (Unauthorized), 404 (Not Found)

### 8.9 Fee Configuration Rules

- `feeLevel` (LOW/MEDIUM/HIGH) is mutually exclusive with manual gas params
- Non-EIP-1559 chains: use `gasPrice` + `gasLimit`
- EIP-1559 chains: use `maxFee` + `priorityFee` + `gasLimit`
- L2 chains (OP, BASE) include additional `l1Fee`
- SCA estimations include ERC-4337 fields: `callGasLimit`, `verificationGasLimit`, `preVerificationGas`
