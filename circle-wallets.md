# Circle Developer Docs — Group 5: Wallets (Complete Summary)

> Compiled from 97 documentation pages at developers.circle.com
> Coverage: Wallets overview, Dev-Controlled, User-Controlled, Modular Wallets, Gas Station, Signing APIs, Compliance Engine, Webhooks, SDKs, Release Notes, and all Quickstarts/How-Tos.

---

## Table of Contents

1. [Wallets Platform Overview](#1-wallets-platform-overview)
2. [Account Types (EOA vs SCA vs MSCA)](#2-account-types-eoa-vs-sca-vs-msca)
3. [Key Management & Security](#3-key-management--security)
4. [Infrastructure Models](#4-infrastructure-models)
5. [Developer-Controlled Wallets](#5-developer-controlled-wallets)
6. [User-Controlled Wallets](#6-user-controlled-wallets)
7. [Modular Wallets (MSCA)](#7-modular-wallets-msca)
8. [Gas Station](#8-gas-station)
9. [Signing APIs](#9-signing-apis)
10. [Batch Operations](#10-batch-operations)
11. [Compliance Engine & Transaction Screening](#11-compliance-engine--transaction-screening)
12. [Webhook Notifications](#12-webhook-notifications)
13. [Monitored Tokens](#13-monitored-tokens)
14. [Unified Wallet Addressing (EVM)](#14-unified-wallet-addressing-evm)
15. [Wallet Upgrades](#15-wallet-upgrades)
16. [Wallets on Solana (Specifics)](#16-wallets-on-solana-specifics)
17. [Blockchain Infrastructure](#17-blockchain-infrastructure)
18. [Supported Blockchains & Currencies](#18-supported-blockchains--currencies)
19. [Transaction Limits & Optimizations](#19-transaction-limits--optimizations)
20. [API Rate Limits](#20-api-rate-limits)
21. [Programmable Wallets Primitives](#21-programmable-wallets-primitives)
22. [Asynchronous States & Statuses](#22-asynchronous-states--statuses)
23. [Blockchain Confirmations](#23-blockchain-confirmations)
24. [Synchronous Errors](#24-synchronous-errors)
25. [Idempotent Requests](#25-idempotent-requests)
26. [Developer Account & Console](#26-developer-account--console)
27. [Server-Side SDKs](#27-server-side-sdks)
28. [Bridge Kit Integration (USDC Bridging with Wallets)](#28-bridge-kit-integration)
29. [Release Notes (2024-2026)](#29-release-notes-2024-2026)

---

## 1. Wallets Platform Overview

Circle Wallets is a Wallet-as-a-Service (WaaS) solution enabling developers to store, send, receive, and spend digital assets on behalf of users without managing raw blockchain infrastructure.

### Three Pillars

1. **Secure Key Management** — Passkeys on user devices or Multi-Party Computation (MPC) keys (user or developer-controlled)
2. **Flexible Account Setup** — Supports Externally Owned Accounts (EOA) and Modular Smart Contract Accounts (SCA/MSCA)
3. **Scalable Blockchain Infrastructure** — Managed nodes, transaction broadcasting, balance reading, event indexing

### Asset Support
- ERC-20, ERC-721, ERC-1155 (EVM chains)
- SPL tokens (Solana)
- Native coins on all supported chains

### Integration Options
- RESTful APIs
- Web SDK, iOS SDK, Android SDK, React Native SDK
- Signing APIs for EVM, Solana, NEAR
- Postman collections for testing

---

## 2. Account Types (EOA vs SCA vs MSCA)

### Externally Owned Accounts (EOA)
- Traditional key-pair accounts (private key + public key = address)
- No creation fees
- Require native tokens for gas on EVM chains
- Gas abstraction available on Solana only
- **Supported on**: All networks (Aptos, Arbitrum, Avalanche, Base, Ethereum, Monad, NEAR, Optimism, Polygon, Solana, Unichain, Arc)
- **Best for**: Simple transfers, mass distribution from single wallet, Solana/non-EVM usage

### Smart Contract Accounts (SCA)
- Controlled by EOA, passkey, or multiple keys — no private keys of their own
- Leverage ERC-4337 account abstraction
- Gas abstraction via Gas Station on EVM
- Batch operations supported
- Deferred creation fees (lazy deployment — contract only deployed on first transaction)
- **Supported on**: Arbitrum, Avalanche, Base, Ethereum, Optimism, Polygon (NOT Aptos, Solana, NEAR)
- **Best for**: Web2-like user experience, embedded wallets

### Modular Smart Contract Accounts (MSCA)
- Extend SCA with ERC-6900 module framework
- Passkey-based signing, custom modules (Address Book, etc.)
- Gas abstraction via Gas Station or custom paymaster
- Batch operations supported
- **Supported on**: Arbitrum, Avalanche, Base, Monad, Optimism, Polygon, Unichain
- **Best for**: Advanced onchain wallets with custom logic (multisig, subscriptions, session keys)

### Comparison Table

| Feature | EOA | SCA | MSCA |
|---------|-----|-----|------|
| Creation Fees | None | Yes (deferred) | Yes (deferred) |
| Gas Abstraction | Solana only | Yes (Gas Station) | Yes (Gas Station or custom) |
| Modules | No | No | Yes (ERC-6900) |
| Batch Operations | No | Yes | Yes |
| Key Flexibility | Single private key | Multiple options | Multiple options (incl. passkeys) |

---

## 3. Key Management & Security

### MPC Architecture

**Developer-Controlled Wallets — 2-of-2 MPC:**
- **Option 1**: Circle hosts both MPC nodes, protected by Entity Secret ciphertext
- **Option 2**: Circle + developer each host one MPC node
- **Option 3**: Developer hosts both nodes with keyguard service

**User-Controlled Wallets — 2-of-2 MPC + Shamir's Secret Sharing:**
- MPC nodes hosted by Circle
- Key shards split across 3 parties: developer, end-user, Circle
- User authentication required before MPC signing occurs
- No single party can access the full private key

**Modular Wallets — Passkey-Based:**
- Key material secured on user device's secure enclave
- WebAuthn standard with secp256r1 elliptic curve
- Syncs via iCloud Keychain (Apple) or Google Password Manager (Android)
- Recovery via ECDSA-based recovery keys

### Entity Secret (Dev-Controlled)
- 32-byte private key generated and stored by developer
- Circle never stores it
- Required for wallet creation, transaction initiation
- Encrypted via RSA using Circle's public key before each API call
- **Never reuse ciphertexts** — each request needs fresh encryption
- Loss of both Entity Secret AND recovery file = permanent lockout

---

## 4. Infrastructure Models

Two fundamental models differing in control allocation:

**User-Controlled Wallets:**
- Users retain full autonomy over private keys
- MPC technology abstracts seed phrases
- Three auth methods: social logins, email, PIN
- SDKs: Web, iOS, Android, React Native

**Developer-Controlled Wallets:**
- Developers manage wallets and keys on behalf of users
- Enable sending/receiving, smart contract interactions, NFT minting
- Users never see blockchain complexity
- Pure API/SDK driven (no client-side SDK required)

---

## 5. Developer-Controlled Wallets

### Entity Secret Management

**Generation:**
```javascript
// Node.js
import { generateEntitySecret } from "@circle-fin/developer-controlled-wallets";
generateEntitySecret();

// Python
from circle.web3 import utils
utils.generate_entity_secret()
```

**Registration:**
```javascript
import { registerEntitySecretCiphertext } from "@circle-fin/developer-controlled-wallets";
const response = await registerEntitySecretCiphertext({
  apiKey: "YOUR_API_KEY",
  entitySecret: "YOUR_32_BYTE_HEX_STRING",
  recoveryFileDownloadPath: ""
});
```

**Rotation**: Provide both current and new ciphertexts. Takes immediate effect. Old secret deprecated. New recovery file generated.

**Reset**: Upload recovery file for authentication, then submit new Entity Secret. Takes immediate effect.

### Wallet Sets
- Hierarchical Deterministic (HD) wallet architecture
- All wallets share the same Entity Secret
- Up to 10 million wallets per set; max 1,000 wallet sets per entity
- EVM: same address across multiple chains within a set

### Quickstart: Create First Wallet

**Setup:**
```bash
mkdir dev-controlled-projects && cd dev-controlled-projects
npm init -y && npm pkg set type=module
npm install @circle-fin/developer-controlled-wallets typescript tsx
```

**Full Flow:**
```typescript
// 1. Register Entity Secret
const entitySecret = crypto.randomBytes(32).toString("hex");
await registerEntitySecretCiphertext({ apiKey, entitySecret, recoveryFileDownloadPath: OUTPUT_DIR });

// 2. Create Wallet Set
const client = initiateDeveloperControlledWalletsClient({ apiKey, entitySecret });
const walletSet = await client.createWalletSet({ name: "Circle Wallet Onboarding" });

// 3. Create Wallet
const wallet = await client.createWallets({
  walletSetId: walletSet.id,
  blockchains: ["ARC-TESTNET"],
  count: 1,
  accountType: "EOA",
});

// 4. Send Transaction
const txResponse = await client.createTransaction({
  blockchain: "ARC-TESTNET",
  walletAddress: wallet.address,
  destinationAddress: secondWallet.address,
  amount: ["5"],
  tokenAddress: "0x3600000000000000000000000000000000000000",
  fee: { type: "level", config: { feeLevel: "MEDIUM" } },
});

// 5. Poll for completion
const terminalStates = new Set(["COMPLETE", "FAILED", "CANCELLED", "DENIED"]);
while (!terminalStates.has(currentState)) {
  await new Promise(resolve => setTimeout(resolve, 3000));
  const poll = await client.getTransaction({ id: txId });
}
```

**Run:** `node --env-file=.env --import=tsx create-wallet.ts`

### Onboarding Users

**Existing Users** — Batch create wallets with `POST /developer/wallets` (up to 200 per request), link via `metadata`/`refId` fields.

**New Users** — Pre-create wallet pool, assign on signup, update Circle records with `PUT /developer/wallets/{id}`.

### Transferring Tokens

**Core API**: `createTransaction()` with parameters:
- `blockchain`, `walletAddress`, `destinationAddress`
- `amount` (array), `tokenAddress`
- `fee: { type: "level", config: { feeLevel: "MEDIUM" } }`

**Verification**: `getWalletTokenBalance()` for balance, `listTransactions()` for history.

### Receiving Inbound Transfers

1. Retrieve wallet address via `GET /wallets`
2. Transfer testnet currency from external wallet/faucet
3. Verify via webhooks (recommended) or polling `GET /transactions`

---

## 6. User-Controlled Wallets

### Authentication Methods

| Method | Onboarding | Signing | Key Management |
|--------|------------|---------|----------------|
| **Social Login** (Google, Apple, Facebook) | OAuth redirect | Confirmation UIs (customizable) | MPC with device secure enclave |
| **Email** | Email + OTP | Confirmation UIs (customizable) | MPC with device secure enclave |
| **PIN** | Set PIN + security questions | PIN/biometric confirmation | PIN-encrypted key, user has sole access |

**Token Management**: User tokens expire after 14 days of inactivity; refresh via `/users/token/refresh`.

### SDK Architecture (Challenge-Based Flow)

1. User initiates action in developer's app
2. Developer calls Circle REST API
3. API returns `challengeId`
4. Developer passes challengeId to client SDK
5. SDK triggers user authentication (PIN entry / confirmation UI)
6. Encrypted input sent to Circle's SDK APIs
7. Circle sends webhook notification of status
8. Developer communicates result to user

### SDKs Available

- **Web SDK**: `@circle-fin/w3s-pw-web-sdk` (npm)
- **iOS SDK**: `CircleProgrammableWalletSDK` (CocoaPods, iOS 13+)
- **Android SDK**: Maven repository (API level 21+)
- **React Native SDK**: GitHub repository

All SDKs encrypt request bodies. User keyshares never leave the device.

### Quickstart: Create Wallets with Social Login (Google OAuth)

**Backend** (`/api/endpoints` route):
- `createDeviceToken` — Exchanges deviceId for verification tokens
- `initializeUser` — Creates user, returns challengeId
- `listWallets` — Retrieves user's wallets
- `getTokenBalance` — Fetches balances

**Frontend Flow**:
1. SDK generates device ID
2. Google OAuth redirect and credential exchange
3. Backend establishes Circle user session
4. SDK completes wallet creation with user approval

### Quickstart: Create Wallets with Email OTP

**Backend**:
- `requestEmailOtp` — `POST /v1/w3s/users/email/token` (returns deviceToken, encryptionKey, otpToken)
- `initializeUser` — `POST /v1/w3s/user/initialize` (returns challengeId)
- `listWallets` / `getTokenBalance`

**Frontend Flow**: Send OTP → Verify OTP → Initialize user → Create wallet

**Email Config**: SMTP credentials (e.g., Mailtrap), custom template with `{{ email.otp }}` placeholder.

### Quickstart: Create Wallets with PIN

**Backend**:
- `createUser` — `POST /v1/w3s/users`
- `getUserToken` — Session credentials
- `initializeUser` — Returns challengeId
- `listWallets` / `getTokenBalance`

**Frontend**: User enters unique ID → Backend creates user → SDK executes challenge with PIN UI.

**Security Note**: PIN alone does not verify identity. Production apps must authenticate users first.

### Sending Outbound Transfers

1. Obtain session token: `POST /users/token` (60-min validity)
2. Get wallet/token info: `GET /wallets`, `GET /wallet/{id}/balances`
3. Estimate fees (optional): `POST /transactions/transfer/estimateFee` (returns low/medium/high)
4. Initiate transfer: `POST /user/transactions/transfer` → returns challengeId
5. User authorizes via SDK (PIN/confirmation UI)
6. Monitor: webhooks or `GET /transactions`

### Receiving Inbound Transfers
Same 3-step process as dev-controlled: get address → transfer → verify.

### Contract Execution
`POST /user/transactions/contractExecution` — returns challengeId, user confirms.

### Signature Requests

Three signing endpoints:
- `POST /user/sign/transaction` (walletId + transaction object or rawTransaction)
- `POST /user/sign/message` (walletId + message, EIP-191)
- `POST /user/sign/typedData` (walletId + data, EIP-712)

All return challengeId → user confirms via SDK.

### Account Recovery
- User answers pre-configured security questions → sets new PIN
- API: `POST /user/pin/restore` → returns challengeId
- **Permanent lockout** if both PIN and security question answers are lost

### PIN Reset
- Requires knowing current PIN
- API: `PUT /user/pin` → returns challengeId
- User enters old PIN, sets new PIN

### Confirmation UIs

Customizable UIs for transfers, contract execution, and signing:
- Title, subtitle, token name, amounts, addresses (auto-masked), fees, buttons
- Available for social login and email auth methods
- **Not available for PIN authentication**
- Can be toggled off in Console (not recommended)

### UI Customization APIs

**Web SDK**: `setThemeColor()` (51 color properties), `setCustomSecurityQuestions()`, `setResource()` (12 resource types), `setLocalizations()` (8 screen types)

**Android SDK**: `TextConfig(text, color, typeface)` with gradient support, `LayoutProvider`, `ViewSetterProvider`, 105+ resource keys

**iOS SDK**: `WalletSdkLayoutProvider` protocol, `CirclePWTheme.json` for colors/fonts, `CirclePWLocalizable.strings` for localization, `ImageStore` for icons

**React Native SDK**: `TextConfig`/`IconTextConfig` classes, `WalletSdk.setTextConfigsMap()`, platform-specific resource files

---

## 7. Modular Wallets (MSCA)

### Architecture
- Built on **Viem** framework
- Compliant with **ERC-6900** (modular smart contracts) and **ERC-4337 v0.7** (account abstraction)
- Passkey-based signing with biometric verification
- **WebAuthn** standard with secp256r1 elliptic curve

### Key Features

**Passkey Integration**: Users set up passkeys via biometrics (Face ID, fingerprints). Keys sync across devices via iCloud Keychain or Google Password Manager.

**Transaction Models**:
- Standard userOps sent to bundlers
- Gasless transactions via Circle Gas Station (`paymaster: true`)
- Batch processing (multiple calls in single userOp)

**Parallel Execution**: 2D nonces (nonce key + nonce value) enable simultaneous independent transactions. Set `nonceKey > 0` to activate.

**Cryptographic Operations**: EIP-191 message signing, EIP-712 typed data signing.

**EIP-1193 Provider**: Standard Ethereum RPC compatibility (`eth_requestAccounts`, `personal_sign`, `eth_signTypedData`).

### Modules

Modules are smart contract extensions for MSCAs:
- Standards: ERC-4337 + ERC-6900
- **Address Book Module**: Restricts transfers to pre-approved addresses (ERC20, ERC721, ERC1155). Available on mainnet/testnet at `0x0000000d81083B16EA76dfab46B0315B0eDBF3d0` (version: circle_6900_v1).
- Custom modules can be developed against the ERC-6900 standard.

### Console Setup

1. Create **Client Key** in Console → API & Client Keys → Client Key
2. Configure **Allowed Domain** (web), Bundle ID (iOS), or Package Name + SHA256 (Android)
3. Configure **Passkey Domain Name** (must match Client Key's Allowed Domain)
4. For mobile: host `/.well-known/apple-app-site-association` (iOS) and `/.well-known/assetlinks.json` (Android)
5. Get **Client URL**: `https://modular-sdk.circle.com/v1/rpc/w3s/buidl`

### Quickstart: Create Wallet & Send Gasless Transaction

**Install:**
```bash
npm install @circle-fin/modular-wallets-core
```

**Step 1 — Create Passkey:**
```javascript
const passkeyTransport = toPasskeyTransport(clientUrl, clientKey)
const credential = await toWebAuthnCredential({
  transport: passkeyTransport,
  mode: WebAuthnMode.Register,
  username: 'your-username'
})
```

**Step 2 — Create Client:**
```javascript
const modularTransport = toModularTransport(clientUrl + "/polygonAmoy", clientKey)
const client = createPublicClient({ chain: polygonAmoy, transport: modularTransport })
```

**Step 3 — Create Smart Account:**
```javascript
const smartAccount = await toCircleSmartAccount({
  client,
  owner: toWebAuthnAccount({ credential })
})
const bundlerClient = createBundlerClient({
  smartAccount, chain: polygonAmoy, transport: modularTransport
})
```

**Step 4 — Send Gasless Transaction:**
```javascript
const USDC_CONTRACT = "0x41e94eb019c0762f9bfcf9fb1e58725bfb0e7582"
const userOpHash = await bundlerClient.sendUserOperation({
  calls: [encodeTransfer(to, USDC_CONTRACT, 100000n)],
  paymaster: true
})
const { receipt } = await bundlerClient.waitForUserOperationReceipt({ hash: userOpHash })
```

**Supported Networks**:
- Mainnets: Arbitrum, Avalanche, Base, Monad, Optimism, Polygon, Unichain
- Testnets: Arbitrum Sepolia, Arc Testnet, Avalanche Fuji, Base Sepolia, Monad Testnet, Optimism Sepolia, Polygon Amoy, Unichain Sepolia

### SDKs

**Web SDK** (`@circle-fin/modular-wallets-core`):
- Transports: `toModularTransport`, `toPasskeyTransport`
- Clients: BundlerClient, PublicClient, RpClient
- Accounts: `toCircleSmartAccount`, `toWebAuthnAccount`
- Providers: ModularWalletsProvider, PaymasterProvider, EIP1193Provider
- Utilities: `encodeTransfer`, `getUserOperationGasPrice`, `registerRecoveryAddress`, `executeRecovery`

**iOS SDK** (Swift Package Manager, `modularwallets-ios-sdk`):
- Same transport/client/account pattern as Web
- `BundlerClient`, `PaymasterClient`, `CircleSmartAccount`, `WebAuthnAccount`
- `encodeTransfer()`, `encodeFunctionData()`, `parseGweiToWei()`
- Supports iOS 13.0+, macOS 12.5+, Xcode 14.1+

**Android SDK** (Gradle, Maven repository):
- Kotlin-first with Java CompletableFuture bridge
- Same architecture: `BundlerClient`, `PaymasterClient`, `CircleSmartAccount`
- `encodeTransfer()`, `encodeFunctionData()`, `encodeContractExecution()`

### Passkey Recovery

1. **Register Recovery**: User registers an EOA (from mnemonic) as recovery key via `registerRecoveryAddress`
2. **Execute Recovery**: If passkey is lost, use mnemonic to recreate EOA, create temporary smart account, register new WebAuthn credential, execute recovery via `executeRecovery`
3. Gas estimation: `estimateRegisterRecoveryAddressGas`, `estimateExecuteRecoveryGas`
4. Users MUST securely store recovery mnemonics

### Dynamic Integration

Circle Modular Wallets can integrate with Dynamic's authentication system:
- Dynamic EOA signer → Circle Smart Account owner
- `primaryWallet → getWalletClient() → walletClientToLocalAccount() → toCircleSmartAccount()`
- Supports gasless transactions with `paymaster: true`

---

## 8. Gas Station

### Problem
Blockchains require native token gas fees, creating friction for users who must source tokens across chains.

### Solution
Circle Gas Station uses **paymasters** (EVM, ERC-4337) and **fee-payers** (Solana) to completely abstract gas.

### Architecture
1. **Gas Sponsor Layer**: Paymaster smart contracts (EVM) / Fee-payer wallets (Solana)
2. **Policy Engine**: Custom rules and sponsorship limits per blockchain
3. **Billing System**: Developer charged via credit card

### Supported Networks (22+)
EVM: Ethereum, Arbitrum, Optimism, Base, Avalanche, Polygon, Monad, Unichain, Arc
Non-EVM: Solana (Mainnet + Devnet), Aptos (Mainnet + Testnet)

### Requirements
- EVM: Wallets must have `accountType: "SCA"` for ERC-4337 compliance
- No special API calls needed — gasless transactions are "no different than any other Programmable Wallet transaction" when policy is configured

### Contract Addresses

**EVM Paymaster** (all chains, mainnet + testnet): `0x7ceA357B5AC0639F89F9e378a1f03Aa5005C0a25`

**Solana**: 5 distinct program addresses across mainnet/devnet as fee-payers.

**Aptos**: 5 separate account addresses on mainnet/testnet.

### Policy Management

- Navigate to Console → Gas Station → Create/Update policy
- **One default policy per network** (only default policies auto-sponsor)
- Required settings: Policy Name, Network
- Optional limits: Max spend/day (USD), Max spend/transaction (USD), Max operations/day, Blocked addresses
- Daily limits reset at 0:00 UTC
- First SCA transaction exempt from per-transaction limit
- Testnets include automatic default policies; **Mainnet requires manual setup**

### Billing
- **5% processing fee** on all invoices
- Bills generated monthly (1st of month) or when $100 threshold reached
- **7-day payment window** from invoice date
- Credit card required on file
- Non-payment → policy paused → no more mainnet sponsorship

### Solana ATA Sponsorship
- Permissioned feature (contact Circle to enable)
- Covers rent deposits for Associated Token Account creation
- Must implement monitoring for abuse (users closing ATAs to reclaim rent)
- Abuse results in account suspension + charges

---

## 9. Signing APIs

### Endpoints

**Developer-Controlled:**
- `POST /developer/sign/message` — Sign plaintext messages
- `POST /developer/sign/typedData` — Sign EIP-712 typed data
- `POST /developer/sign/transaction` — Sign raw transactions
- `POST /developer/sign/delegateAction` — Delegate actions (NEAR)

**User-Controlled:**
- `POST /user/sign/message`
- `POST /user/sign/typedData`
- `POST /user/sign/transaction`

### Supported Chains
- **EVM**: All supported EVM chains + generic `EVM`/`EVM-TESTNET` for unsupported chains
- **Solana**: Message signing only (no typed data)
- **NEAR**: Developer-controlled only

### EVM Transaction Signing

**Wallet Creation**: Use `blockchains: ["EVM-TESTNET"]` with `accountType: "EOA"`.

**Transaction Building** (using viem):
```javascript
const nonce = await client.getTransactionCount({ address })
const gasLimit = await client.estimateGas({ to, value, data })
// Construct EIP-1559 transaction with maxFeePerGas, maxPriorityFeePerGas, chainId
```

**Signing**:
```javascript
const response = await circleDeveloperSdk.signTransaction({
  walletID: "<wallet-id>",
  transaction: txObjectString,
});
```

**Warning**: Signing with generic EVM wallet when a native wallet exists on that chain may cause stuck transactions.

### Solana Transaction Signing

**Raw transaction must be Base64 encoded.**

```javascript
// Build and serialize transaction
const rawTransaction = transaction.serialize({ requireAllSignatures: false });
const base64Tx = rawTransaction.toString("base64");

// Sign
const response = await circleDeveloperSdk.signTransaction({
  walletId: "SOL-WALLET-ID",
  rawTransaction: base64Tx,
});

// Broadcast
const signed = Buffer.from(response.data.signedTransaction, "base64");
await connection.sendRawTransaction(signed);
```

### NEAR Transaction Signing

- Wallet must be activated by sending NEAR tokens first
- Transaction built using `near-api-js`, serialized with Borsh, encoded to Base64
- Circle handles signing; developer broadcasts

### EVM Testnet Chain IDs
130+ testnet chains supported for signing, including:
- Sepolia (11155111), Amoy (80002), Avalanche Fuji (43113)
- Arbitrum Sepolia (421614), Base Sepolia (84532)
- BNB Testnet (97), Fantom Testnet (4002), Linea Testnet (59140)

---

## 10. Batch Operations

Available for SCA and MSCA wallets via the `contractExecution` endpoint.

### Advantages
- Single atomic transaction (all-or-nothing)
- Gas efficiency (cheaper than individual transactions)
- Single user confirmation

### Implementation

**Function Signature**: `executeBatch((address, uint256, bytes)[])`

Each operation: `(contractAddress, nativeTokenAmount, encodedFunctionCallData)`

**Example — Batch USDC Transfers**:
```javascript
// Encode two transfer calls using ethers.js
const call1 = iface.encodeFunctionData("transfer", [addr1, amount1])
const call2 = iface.encodeFunctionData("transfer", [addr2, amount2])
// Submit as batch via contractExecution endpoint
```

**Example — CCTP Batch (approve + depositForBurn)**:
Batch an ERC-20 `approve` call followed by `depositForBurn` for cross-chain token burning.

---

## 11. Compliance Engine & Transaction Screening

### Overview
Real-time compliance tool for AML/CTF requirements. Flags high-risk transactions before execution.

### Transaction Screening Methods

**Embedded Screening**: Automatic within programmable wallet transaction flow. Results in `transactionScreeningEvaluation` field of Get Transaction response.

**Standalone Screening**: `POST /v1/w3s/compliance/screening/addresses`
```javascript
{
  idempotencyKey: "uuid",
  address: "0x...",
  chain: "ETH-SEPOLIA"
}
```

### Response Structure
- `result`: APPROVED or DENIED
- `decision`: ruleName + actions (DENY, REVIEW, FREEZE_WALLET, APPROVE)
- Risk details: riskScore, riskCategories
- Vendor response (testnet only)

### Rule Management

**Rule Types**: Restrictive (block/freeze) and Alert-only (notifications).

**Rule Components**: Name, Description, Actions, Criteria (Risk Category + Risk Score + Risk Type).

**Default Mandatory Rules**:
1. Circle's Sanctions Blocklist (OFAC)
2. Your blocklist (custom)
3. Frozen (wallet freeze enforcement)
4. Your allowlist (custom)

**Risk Categories**: Sanctions, Terrorist Financing, CSAM, PEP, Gambling, Illicit Behavior, High Risk Industry, Other.

### Alert Management
- Console: Compliance Engine → Alerts
- Actions: Freeze wallet, Add to blocklist, Close/manage alert status
- Links to wallet and transaction detail pages

### Testing with Magic Values

| Address Suffix | Rule Triggered | Action |
|---------------|----------------|--------|
| 9999 | Circle's Sanctions Blocklist | DENY (outbound), REVIEW (inbound) |
| 8888 | Frozen User Wallet | DENY, REVIEW |
| 7777 | Your blocklist | DENY, REVIEW |
| 8999 | Severe Sanctions Risk | FREEZE_WALLET, DENY |
| 8899 | Severe Terrorist Financing | FREEZE_WALLET, DENY |
| 8889 | Severe CSAM Risk | FREEZE_WALLET, DENY |
| 7779 | Severe Illicit Behavior | FREEZE_WALLET, DENY |
| 7666 | High Illicit Behavior | REVIEW |
| 7766 | High Gambling Risk | REVIEW |

**Availability**: Testnet + Mainnet, eligible customers only (request access).

---

## 12. Webhook Notifications

### Setup
1. Expose HTTPS endpoint handling HEAD + POST requests
2. Register in Console → Webhooks → Add a Webhook
3. Test with webhook.site during development

### Payload Structure
```json
{
  "subscriptionId": "uuid",
  "notificationId": "uuid",
  "notificationType": "transactions.complete",
  "notification": { ... },
  "timestamp": "ISO-8601",
  "version": 2
}
```

### Security — Digital Signature Verification
- Headers: `X-Circle-Signature` (Base64 ECDSA) + `X-Circle-Key-Id`
- Retrieve public key: `GET /v2/notifications/publicKey/{keyId}`
- Algorithm: ECDSA with SHA-256

### Operational Constraints
- **5-second response timeout**
- Rate limit: 20 notifications per environment
- Authorized IPs: 54.243.112.156, 100.24.191.35, 54.165.52.248, 54.87.106.46
- Process async — do not block the response loop

### Transaction Notification Types

| Type | State | Description |
|------|-------|-------------|
| Inbound | CONFIRMED | On blockchain, pending final confirmations |
| Inbound | COMPLETED | Fully confirmed, funds available |
| Outbound | QUEUED | Initiated, not yet processed |
| Outbound | SENT | Transmitted to blockchain node |
| Outbound | CONFIRMED | Broadcast, awaiting confirmations |
| Outbound | COMPLETED | Fully settled |
| Outbound | CANCELED | Cancellation confirmed |
| Outbound | FAILED | Rejected (insufficient balance, etc.) |

### Challenge Notifications
- COMPLETE — User passed challenge, action proceeds
- FAILED — Challenge unsuccessful, re-initiate required

### Webhook Logs
- Console → Webhook Logs tab
- Shows delivery status, log ID, notification ID, event type, payload
- Resend capability for troubleshooting

---

## 13. Monitored Tokens

Filter tokens visible in API responses per developer preference.

### API Endpoints
- `POST config/entity/monitoredTokens` — Add tokens
- `POST /config/entity/monitoredTokens/delete` — Remove tokens
- `PUT /config/entity/monitoredTokens` — Replace entire list
- `PUT /config/entity/monitoredTokens/scope` — Set scope (SELECTED or MONITOR_ALL)

### Default Behavior
No monitored tokens = all tokens monitored. Override with `includeAll: true` in balance/transaction queries.

### Supported Standards
ERC-20, ERC-721, ERC-1155, native coins across 14+ blockchains.

---

## 14. Unified Wallet Addressing (EVM)

Same address across multiple EVM blockchains within a wallet set.

### Developer-Controlled Implementation

**Batch Creation**: `POST /wallets` with `walletSetId`, array of `blockchains`, array of `refIds`. All EVM wallets sharing a `refId` in one API call get identical addresses.

**On-Demand Derivation**: `PUT /wallets/{id}/blockchains/{blockchain}` generates wallet on new chain preserving original address.

**Unsupported Chain Recovery**: Create wallet with `blockchain: EVM`, use Sign Transaction API, broadcast yourself.

### Address Derivation
- Derived from wallet set indexes (e.g., `0x3`)
- Single-chain creation starts at index `0x1`
- Multi-chain simultaneous creation uses latest available index across all chains

### Limits
1,000 wallet sets total, up to 10 million wallets per set.

---

## 15. Wallet Upgrades

### Architecture
ERC-1967 proxy pattern — upgrade logic contract while maintaining wallet address.

### API Endpoints
- Developer-controlled: `POST /developer/transactions/walletUpgrade`
- User-controlled: `POST /user/transactions/walletUpgrade` (requires challenge)

### Supported Paths

| Source | Destination | Function |
|--------|-------------|----------|
| circle_4337_v1 | circle_6900_singleowner_v2 | `upgradeToAndCall(address,bytes)` |
| circle_6900_singleowner_v1 | circle_6900_singleowner_v2 | `upgradeTo(address)` |

### Requirements
- circle_4337_v1 wallets must be **lazy deployed** first (via outbound transfer or contract execution)
- Testnets: send native tokens (~300,000 gas) to wallet owner address
- Mainnet: tokens airdropped for upgrade fees

---

## 16. Wallets on Solana (Specifics)

- **EOA only** — SCA not supported
- No transaction replacement (no accelerate/cancel)
- Priority fees only, no max fee concept
- Default gas limit: 200,000 micro-lamports
- Supports message signing only (no typed data)
- ATAs auto-created for recipients; arbitrary token accounts NOT supported
- Gas Station via `feePayer` parameter
- Fee estimation returns same values for all levels (low/medium/high)

---

## 17. Blockchain Infrastructure

### Services Provided
- **Broadcasting**: Circle handles transaction broadcasting (or use Signing APIs for custom node providers)
- **Indexing**: Real-time transfer event and balance indexing with low-latency APIs
- **Gas Station**: Native token abstraction via policy management
- **Bundler**: ERC-4337 smart contract account bundling

### Full Infrastructure Support
Aptos, Arbitrum, Avalanche, Base, Ethereum, Monad, Optimism, Polygon, Solana, Unichain

### Signing-Only Support (BYO Infrastructure)
NEAR, other EVM chains (via `EVM`/`EVM-TESTNET`)

---

## 18. Supported Blockchains & Currencies

### Network Support Matrix

| Network | Chain Code | EOA | SCA | Gas Station | MSCA |
|---------|-----------|-----|-----|-------------|------|
| Ethereum | ETH | Yes | Yes | Yes | No |
| Arbitrum | ARB | Yes | Yes | Yes | Yes |
| Avalanche | AVAX | Yes | Yes | Yes | Yes |
| Base | BASE | Yes | Yes | Yes | Yes |
| Optimism | OP | Yes | Yes | Yes | Yes |
| Polygon | MATIC | Yes | Yes | Yes | Yes |
| Monad | MONAD | Yes | No | Yes | Yes |
| Unichain | UNI | Yes | No | Yes | Yes |
| Solana | SOL | Yes | No | Yes | No |
| Aptos | APTOS | Yes | No | Yes | No |
| NEAR | NEAR | Yes (signing only) | No | No | No |
| Generic EVM | EVM | Yes (signing only) | No | No | No |

### Token Standards
- EVM: Native coin, ERC-20, ERC-721, ERC-1155
- Solana: Native coin, SPL tokens

---

## 19. Transaction Limits & Optimizations

### Queue Limits by Blockchain

| Chain | EOA Limit | SCA Limit |
|-------|-----------|-----------|
| Ethereum | 32 | 1 |
| Polygon | 16 | 1 |
| Avalanche | 32 | 1 |
| Solana | Unlimited | N/A |

Error code 155264 when exceeded.

### Optimization Strategies
1. Set `fee_level` to MEDIUM or HIGH for priority
2. Use webhook notifications for real-time monitoring
3. Cancel underperforming transactions via cancel API
4. Accelerate stuck transactions (applies HIGH fee)
5. Deploy multiple sender wallets to bypass per-wallet limits
6. Use batch operations for SCA wallets

### Transaction States
INITIATED → CLEARED → QUEUED → SENT → (STUCK) → CONFIRMED → COMPLETE
Terminal states: COMPLETE, FAILED, CANCELLED, DENIED

---

## 20. API Rate Limits

| Operation Type | Rate Limit |
|---------------|------------|
| Standard POST | 5 RPS |
| Elevated POST (wallet creation, signing, fee estimation, etc.) | 10 RPS |
| GET requests | 20 RPS |

---

## 21. Programmable Wallets Primitives

### Core Entities

**Wallets**: Address, Custody Type (user/developer), Account Type (EOA/SCA), Version, State (active/frozen), Blockchain.

**Wallet Sets**: HD wallet architecture, up to 10M wallets/set, 1000 sets/entity, unified EVM addressing.

**Users**: userId (unique), userToken (60-min session), PIN auth, security questions, challenge checkpoints.

**Smart Contracts**: Deployed/imported contracts with ABI, function signatures, events.

**Transactions**: TRANSFER, CONTRACT_EXECUTION, CONTRACT_DEPLOY operations with state tracking.

**Gas Station Policies**: Network, daily spend cap (USD), per-transaction cap, operations/day, blocklist.

**Monitored Tokens**: Selective token filtering (native, ERC-20, ERC-721, ERC-1155).

---

## 22. Asynchronous States & Statuses

### Transaction States

| State | Description | Cancelable | Acceleratable |
|-------|-------------|------------|---------------|
| INITIATED | Transaction created | Yes | No |
| QUEUED | In processing queue | Yes | No |
| CLEARED | Passed risk screening | No | No |
| SENT | In mempool, has txHash | Yes | Yes |
| STUCK | Sent but not mined | No | Yes |
| CONFIRMED | In a mined block | No | No |
| COMPLETE | Fully confirmed (terminal) | No | No |
| CANCELLED | Cancelled (terminal) | No | No |
| FAILED | Failed with errorReason (terminal) | No | No |
| DENIED | Platform denied (terminal) | No | No |

### Challenge Statuses
PENDING → IN_PROGRESS → COMPLETED / FAILED / EXPIRED

---

## 23. Blockchain Confirmations

Confirmations = number of block validations before finality.

### Two API States
- **CONFIRMED**: Transaction in a block, balance updated
- **COMPLETED**: Confirmation threshold reached, irreversible

### Confirmation Numbers

| Chain | Confirmations | Approximate Time |
|-------|---------------|-----------------|
| Ethereum | 12 | ~3 min |
| Arbitrum | 12 ETH blocks | ~3 min |
| Solana | 33 | ~13 sec |
| Avalanche | 1 | ~2 sec |
| Polygon PoS | 50 | ~2 min |

---

## 24. Synchronous Errors

### Key Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| 155501 | 409 | Frozen wallet — read-only operations only |
| 155502 | 403 | Max wallet capacity per set reached (~1M) |
| 155505 | 400 | SCA wallet requires first tx before additional txs |
| 155507 | 400 | SCA not supported on given blockchain |
| 155508 | 400 | Cannot create multiple user-controlled SCA wallets across chains simultaneously |
| 155509 | 400 | Mainnet Paymaster policy required before SCA creation |
| 155601 | 400 | Wallet set already exists |

---

## 25. Idempotent Requests

- All Circle APIs support idempotent requests
- Provide unique `idempotencyKey` as UUID v4
- Subsequent requests with same key return same result without re-processing
- Essential for safe retry logic in payment/transaction workflows

---

## 26. Developer Account & Console

### Account Setup
- Sign up at console.circle.com/signup
- Dashboard: API keys, client keys, activity notifications, API logs

### API Key Types
- **Standard**: Full read/write to all APIs
- **Restricted Access**: Customizable permissions per product (Webhooks, Wallets, Contracts) with read-only or read/write levels
- IP allowlist support for security
- Testnet prefix: `TEST_API_KEY:`, Mainnet: `LIVE_API_KEY:`

### Client Keys
- Bound to specific host domain (web), bundle ID (iOS), or package name (Android)
- Required for modular wallets SDK calls
- Create separate keys per platform

### Sandbox vs Production
- Replace `TEST_API_KEY` with `LIVE_API_KEY` to switch
- Mainnet: longer confirmation times, real gas fees
- Upgrade: Console → "UPGRADE TO PROD" → verify info → accept terms

### Faucet
- Console: console.circle.com/faucet
- 10 requests per account per 24 hours
- USDC: 20 per request
- Native tokens: APT (0.01), ARB (0.01), AVAX (0.1), ETH (0.01), SOL (0.01), etc.
- Programmatic: `POST /v1/faucet/drips` (requires mainnet upgrade)

### Developer Account Logs
- 7-day retention
- Captures: HTTP status, path, request ID, user agent, idempotency key, timestamps, request/response bodies
- Filterable by search, date range, status, HTTP method, path

### Team Management
- **Owner**: Full control, cannot be removed
- **Admin**: Full control including team management
- **View-only**: Read access, no modifications
- Invitations expire in 7 days

### Postman Collections
- Two collections: Wallets and Contracts
- Variables: `baseUrl` (`https://api.circle.com/v1/w3s`), `hex-encoded-entity-secret`, `entity-public-key`, `X-user-token`, `walletId`, `walletSetId`
- Auto re-encrypts entity secret per request
- Run "Get public key for entity" first to initialize

### Compliance Requirements
- Must not transact with OFAC-sanctioned addresses
- Violation → wallet restricted from outbound transactions
- Remediation: contact Circle support for quarantine wallet

---

## 27. Server-Side SDKs

### Node.js SDK (Developer-Controlled)
- Package: `@circle-fin/developer-controlled-wallets`
- Init: `initiateDeveloperControlledWalletsClient({ apiKey, entitySecret })`
- Methods: `createWalletSet()`, `createWallets()`, `createTransaction()`, `getTransaction()`, `listWallets()`, `getWalletTokenBalance()`
- Requires Node.js v22+

### Python SDK (Developer-Controlled)
- Package: `circle-developer-controlled-wallets` (pip)
- Init: `utils.init_developer_controlled_wallets_client(api_key, entity_secret)`
- Config: api_key, entity_secret, host (optional), user_agent (optional)
- Pydantic v2 from version 9+

### Node.js SDK (User-Controlled)
- Package: `@circle-fin/user-controlled-wallets`
- Init: `initiateUserControlledWalletsClient({ apiKey })`
- Methods: `createTransaction()` → returns challengeId
- Requires Node.js v22+

### Python SDK (User-Controlled)
- Package: `circle-user-controlled-wallets` (pip)
- Init: `utils.init_user_controlled_wallets_client(api_key)`
- Operations: `create_user()`, `get_user()`, `get_user_token()`
- Pydantic v2 from version 9+

---

## 28. Bridge Kit Integration

### Bridging USDC with Circle Wallets Adapter

**Install:**
```bash
npm install @circle-fin/bridge-kit @circle-fin/adapter-circle-wallets typescript tsx dotenv
```

**Implementation:**
```typescript
const result = await kit.bridge({
  from: { adapter, chain: "Solana_Devnet", address: process.env.SOLANA_WALLET_ADDRESS },
  to: { adapter, chain: "Arc_Testnet", address: process.env.EVM_WALLET_ADDRESS },
  amount: "1.00"
});
```

**Prerequisites**: Developer-controlled wallets on source/destination chains, testnet USDC, native tokens.

---

## 29. Release Notes (2024-2026)

### 2024 Highlights
- **Solana Integration** (June) — Full programmable wallet support
- **Arbitrum Support** (September) — All wallet/transaction/contract operations
- **EVM Generic Wallets** (October) — `EVM`/`EVM-TESTNET` for signing on 130+ chains
- **Unichain** (November) — Testnet support
- **Signing APIs** — Solana (July-Oct), EVM (October), NEAR (Q3)
- **User Auth Enhancements** (July) — Social login, email, OTP endpoints
- **Event Monitoring** (October) — Contract monitors and events API
- **Transaction Screening** (September) — Compliance engine address screening
- **Travel Rule** (December, beta) — Eligibility, validity, PII exchange endpoints
- **Ramp/USDC Access** (October) — Sessions, quotes, trade configurations

### 2025 Highlights
- **Monad Support** (November)
- **Avalanche SCA/MSCA/Gas Station** (July)
- **Aptos Integration** (June)
- **Optimism & Base Mainnet** (April)
- **Unichain Mainnet** (February)
- **Wallet Derivation API** (April) — Cross-chain wallet derivation
- **Modular Wallets SDK** — Gas price optimization, passkey recovery functions
- **User-Controlled EVM Signing** (January) — Confirmation UIs for signing
- **Batch Operations** (January) — Enhanced documentation
- **Sorting Parameters** (July) — ASC/DESC for list endpoints
- **Fee Response Fields** (August) — `networkFeeRaw` added

### 2026 Highlights
- **Solana ATA Sponsorship** (January 28) — Gas Station covers ATA rent deposits
- **Quickstart Overhaul** (January 20) — Social login, email OTP, PIN guides restructured for end-to-end clarity

---

## Gas Fees Reference

### Fee Configuration Options

**Simple (Fee Levels):**
```json
{ "type": "level", "config": { "feeLevel": "MEDIUM" } }
```
Options: `LOW`, `MEDIUM`, `HIGH`

**Advanced (Gas Limits):**
```json
{
  "type": "gas",
  "config": {
    "gasLimit": "...",
    "priorityFee": "...",
    "maxFee": "..."
  }
}
```

### EVM Fee Formula
`(base fee + priority fee) x gas used`

- Base fee: Network-determined, burned
- Priority fee: Validator tip
- Max fee: Upper bound protection
- Note: Arbitrum does not support priority fees

### Solana Fee Model
- Priority fees only (no max fee concept)
- Default: 200,000 micro-lamports when unspecified
- Fee estimation returns identical values across all levels

---

## Quick Reference: Key API Endpoints

### Developer-Controlled Wallets
| Endpoint | Purpose |
|----------|---------|
| `POST /developer/wallets` | Create wallets (up to 200/request) |
| `GET /wallets` | List wallets |
| `POST /developer/transactions/transfer` | Transfer tokens |
| `POST /developer/transactions/contractExecution` | Execute contracts |
| `POST /developer/sign/message` | Sign message |
| `POST /developer/sign/typedData` | Sign typed data |
| `POST /developer/sign/transaction` | Sign raw transaction |
| `POST /developer/transactions/walletUpgrade` | Upgrade wallet |

### User-Controlled Wallets
| Endpoint | Purpose |
|----------|---------|
| `POST /users` | Create user |
| `POST /users/token` | Get session token (60-min) |
| `POST /user/initialize` | Initialize user wallet |
| `POST /user/transactions/transfer` | Transfer tokens (returns challengeId) |
| `POST /user/transactions/contractExecution` | Execute contract (returns challengeId) |
| `POST /user/sign/message` | Sign message |
| `POST /user/sign/typedData` | Sign typed data |
| `POST /user/pin/restore` | Recover account |
| `PUT /user/pin` | Reset PIN |

### Common
| Endpoint | Purpose |
|----------|---------|
| `GET /transactions` | List transactions |
| `GET /wallets/{id}/balances` | Get token balances |
| `POST /transactions/transfer/estimateFee` | Estimate transfer fee |
| `POST /compliance/screening/addresses` | Screen address |
| `POST /config/entity/monitoredTokens` | Add monitored tokens |

---

*End of Group 5 Wallets Summary — 97 pages covered.*
