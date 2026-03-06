# Circle Cross-Chain Documentation - Comprehensive Summary

> Generated from 89 developer documentation pages covering CCTP v2, CCTP v1, Bridge Kit, Gateway, Nanopayments, and the x402 protocol.

---

## Table of Contents

1. [CCTP v2 (Cross-Chain Transfer Protocol)](#1-cctp-v2-cross-chain-transfer-protocol)
2. [CCTP v1 (Legacy)](#2-cctp-v1-legacy)
3. [Bridge Kit SDK](#3-bridge-kit-sdk)
4. [Gateway](#4-gateway)
5. [Nanopayments & x402 Protocol](#5-nanopayments--x402-protocol)
6. [Release Notes (2026)](#6-release-notes-2026)

---

## 1. CCTP v2 (Cross-Chain Transfer Protocol)

### 1.1 Overview

CCTP is a **permissionless onchain utility** that facilitates native USDC transfers across blockchains via a burn-and-mint mechanism. USDC is burned on the source chain and minted (1:1) on the destination chain -- no liquidity pools, no wrapped tokens.

**Key V2 Features:**
- **Fast Transfer**: ~8-20 seconds (uses soft finality)
- **Standard Transfer**: ~15-19 minutes for Ethereum/L2s (uses hard finality)
- **Programmable Hooks**: Arbitrary data attached to burn messages, enabling custom logic execution on destination chains
- **Forwarding Service**: Circle handles destination chain mint transaction automatically
- **Per-transaction limit**: $10 million USDC per single CCTP transaction

### 1.2 Architecture & Message Flow

The protocol operates through a three-step process:

1. **Source Domain**: User calls `TokenMessengerV2.depositForBurn()` which burns USDC and emits a message via `MessageTransmitterV2.sendMessage()`
2. **Attestation Service (Iris)**: Circle's offchain service signs the message after sufficient block confirmations
3. **Destination Domain**: Consumer calls `MessageTransmitterV2.receiveMessage()` with the attestation to mint USDC

**Core Contracts (EVM):**
| Contract | Purpose |
|----------|---------|
| `TokenMessengerV2` | Entry point for cross-chain USDC transfers |
| `MessageTransmitterV2` | Generic message passing infrastructure |
| `TokenMinterV2` | Manages USDC minting/burning per chain |
| `MessageV2` | Address conversion utilities (bytes32 <-> address) |

### 1.3 Key Functions

**`depositForBurn(amount, destinationDomain, mintRecipient, burnToken, destinationCaller, maxFee, minFinalityThreshold)`**
- Burns USDC on source chain
- `destinationCaller`: restricts who can execute `receiveMessage` on destination (bytes32(0) = anyone)
- `maxFee`: maximum acceptable fee in USDC subunits
- `minFinalityThreshold`: 1000 = Fast Transfer, 2000 = Standard Transfer

**`depositForBurnWithHook(..., hookData)`**
- Same as above with additional `hookData` bytes for destination-side custom logic
- CCTP treats hooks as opaque metadata -- integrators control execution

**`receiveMessage(message, attestation)`**
- Receives attested message on destination chain
- Messages with a given nonce can only be broadcast once (idempotent)

**`getMinFeeAmount(amount)`** -- calculates minimum Standard Transfer fee

### 1.4 Message Format

**Header (148 bytes fixed):**

| Field | Offset | Type | Description |
|-------|--------|------|-------------|
| version | 0 | uint32 | CCTP version (1) |
| sourceDomain | 4 | uint32 | Origin domain ID |
| destinationDomain | 8 | uint32 | Target domain ID |
| nonce | 12 | bytes32 | Unique message nonce |
| sender | 44 | bytes32 | Source chain caller |
| recipient | 76 | bytes32 | Destination handler |
| destinationCaller | 108 | bytes32 | Permitted caller or bytes32(0) |
| minFinalityThreshold | 140 | uint32 | Finality level |
| finalityThresholdExecuted | 144 | uint32 | Actual finality used |
| messageBody | 148 | bytes | Dynamic payload |

**BurnMessageV2 Body:**

| Field | Offset | Type | Description |
|-------|--------|------|-------------|
| version | 0 | uint32 | Version (1) |
| burnToken | 4 | bytes32 | Source token address |
| mintRecipient | 36 | bytes32 | Recipient on destination |
| amount | 68 | uint256 | Burned amount |
| messageSender | 100 | bytes32 | Original caller |
| maxFee | 132 | uint256 | Max fee cap |
| feeExecuted | 164 | uint256 | Actual fee charged |
| expirationBlock | 196 | uint256 | 24hr expiration block |
| hookData | 228 | bytes | Arbitrary data |

### 1.5 Finality & Block Confirmations

**Fast Transfer (minFinalityThreshold <= 1000):**
- Ethereum, Starknet: 2-4 blocks (~20 seconds)
- Most L2s (Arbitrum, Base, OP, etc.): 1 block (~8 seconds)
- Solana: 2-3 blocks (~8 seconds)

**Standard Transfer (minFinalityThreshold >= 2000):**
- Ethereum, most OP Stack chains: ~65 ETH blocks (15-19 minutes)
- Solana: 32 blocks (~25 seconds)
- Polygon PoS, BNB Smart Chain: 2-3 blocks (~2-8 seconds)
- Starknet: ~65 ETH blocks (4-8 hours)
- Linea: 1 block (6-32 hours)

### 1.6 Fees

- **Standard Transfer**: FREE (0% across all chains)
- **Fast Transfer**: 0-14 basis points depending on source chain (e.g., $0-$1.40 per $1,000)
- Fees deducted from burn amount on source chain
- `maxFee` parameter protects against unexpected charges; transaction reverts if fee exceeds it

**Fee API:**
```
GET /v2/burn/USDC/fees/{sourceDomainId}/{destDomainId}
```
Response:
```json
[
  { "finalityThreshold": 1000, "minimumFee": 1 },  // Fast: 1 basis point
  { "finalityThreshold": 2000, "minimumFee": 0 }   // Standard: free
]
```

**Fee Calculation:**
1. Convert amount to subunits (multiply by 10^6)
2. Get `minimumFee` from API (in basis points)
3. `protocolFee = (amount * minimumFee * 100) / 1,000,000`
4. Apply 10-20% buffer: `maxFee = protocolFee * 1.2`

### 1.7 Fast Transfer Allowance

A global liquidity pool backing all in-process Fast Transfers. When allowance is depleted, Fast Transfers become unavailable until replenishment via hard finality completion.

**API:**
```
GET /v2/fastBurn/USDC/allowance
```
Response:
```json
{
  "allowance": 99999999225.24174,
  "lastUpdated": "2025-12-02T13:17:02.453Z"
}
```

### 1.8 Forwarding Service

Automates destination chain minting -- Circle handles attestation fetching, gas payment, and mint submission.

**How to use:**
- Include forward request data in source chain burn transaction's `hookData`
- Magic bytes: `cctp-forward` (24 bytes) + version uint32 (0) + data length uint32 (0)
- Static hex for basic forwarding: `0x636374702d666f72776172640000000000000000000000000000000000000000`

**Service Fees:**
- Ethereum destination: $1.25 per transfer
- Other chains: $0.20 per transfer

**Fee Priority:** If `maxFee` is insufficient for both Fast Transfer + Forwarding, the system prioritizes forwarding and downgrades to Standard Transfer.

### 1.9 Supported Chains & Domains (V2)

| Domain | Blockchain | Fast Transfer | Forwarding |
|--------|-----------|:---:|:---:|
| 0 | Ethereum | Yes | Yes |
| 1 | Avalanche | No | No |
| 2 | OP Mainnet | Yes | Yes |
| 3 | Arbitrum | Yes | Yes |
| 5 | Solana | Yes | No |
| 6 | Base | Yes | Yes |
| 7 | Polygon PoS | No | No |
| 10 | Unichain | Yes | Yes |
| 11 | Linea | Yes | Yes |
| 12 | Codex | Yes | No |
| 13 | Sonic | No | No |
| 14 | World Chain | Yes | Yes |
| 15 | Monad | No | No |
| 16 | Sei | No | No |
| 17 | BNB Smart Chain | Yes | No |
| 18 | XDC | No | No |
| 19 | HyperEVM | No | No |
| 21 | Ink | Yes | Yes |
| 22 | Plume | Yes | No |
| 25 | Starknet | Yes | No |
| 26 | Arc Testnet | No | No |

**Token Support:**
- USDC: All domains except BNB Smart Chain
- USYC: Ethereum and BNB Smart Chain only

### 1.10 Contract Addresses

**Mainnet (V2) -- All EVM chains share identical addresses:**
- TokenMessengerV2: `0x28b5a0e9C621a5BadaA536219b3a228C8168cf5d`
- MessageTransmitterV2: `0x81D40F21F12A8F0E3252Bccb954D722d4c464B64`
- TokenMinterV2: `0xfd78EE919681417d192449715b2594ab58f5D002`
- MessageV2: `0xec546b6B005471ECf012e5aF77FBeC07e0FD8f78`

**Testnet (V2) -- All chains share identical addresses:**
- TokenMessengerV2: `0x8FE6B999Dc680CcFDD5Bf7EB0974218be2542DAA`
- MessageTransmitterV2: `0xE737e5cEBEEBa77EFE34D4aa090756590b1CE275`
- TokenMinterV2: `0xb43db544E2c27092c107639Ad201b3dEfAbcF192`
- MessageV2: `0xbaC0179bB358A8936169a63408C8481D582390C4`

**Solana Programs (V2):**
- MessageTransmitterV2: `CCTPV2Sm4AdWt5296sk4P66VBZ7bEhcARwFaaS9YPbeC`
- TokenMessengerMinterV2: `CCTPV2vPZJS2u2BBsUoscuikbYjnpFmbFsvVuJdgUMQe`

**Starknet Contracts (V2):**

Mainnet:
- TokenMessengerMinterV2: `0x07d421B9cA8aA32DF259965cDA8ACb93F7599F69209A41872AE84638B2A20F2a`
- MessageTransmitterV2: `0x02EBB5777B6dD8B26ea11D68Fdf1D2c85cD2099335328Be845a28c77A8AEf183`

Testnet:
- TokenMessengerMinterV2: `0x04bDdE1E09a4B09a2F95d893D94a967b7717eB85A3f6dEcA8c080Ee01fBc3370`
- MessageTransmitterV2: `0x04db7926C64f1f32a840F3Fa95cB551f3801a3600Bae87aF87807A54DCE12Fe8`

### 1.11 HyperCore Integration

CCTP enables USDC transfers to HyperCore (Hyperliquid) via a two-step process: standard CCTP to HyperEVM, then forwarding to HyperCore via `CoreDepositWallet`.

**Specialized Contracts:**

| Contract | Chain | Mainnet Address | Testnet Address |
|----------|-------|----------------|----------------|
| CctpExtension | Arbitrum | `0xA95d9c1F655341597C94393fDdc30cf3c08E4fcE` | `0x8E4e3d0E95C1bEC4F3eC7F69aa48473E0Ab6eB8D` |
| CctpForwarder | HyperEVM | `0xb21D281DEdb17AE5B501F6AA8256fe38C4e45757` | `0x02e39ECb8368b41bF68FF99ff351aC9864e5E2a2` |
| CoreDepositWallet | HyperEVM | `0x6B9E773128f453f5c2C60935Ee2DE2CBc5390A24` | `0x0B80659a4076E9E93C7DbE0f10675A16a3e5C206` |

**CoreDepositWallet Functions:**
- `deposit(amount, destinationDex)` -- deposits from caller; 0=perps, uint32.max=spot
- `depositFor(recipient, amount, destinationId)` -- third-party deposits
- `depositWithAuth(amount, ..., v, r, s, destinationDex)` -- ERC-3009 authorization-based

**Warning:** Sending USDC directly to CoreDepositWallet address does NOT trigger deposit -- funds are permanently stuck.

**HyperCore Constraints:**
- Balances are protocol-level credits, not Circle-issued USDC
- New account creation costs 1 USDC fee
- Deposits under 1 USDC to new accounts fail
- Testnet recipients must exist on mainnet (capped at $1,000)

**Hook Data for HyperCore Forwarding:**
```
Magic bytes (24): "cctp-forward" + padding
Version (4 bytes): uint32(0)
Data length (4 bytes): uint32(24)
Recipient (20 bytes): HyperCore address
Destination DEX (4 bytes): 0=perps, 0xFFFFFFFF=spot
```

### 1.12 Attestation & Verification

**API Endpoints:**

| Endpoint | Purpose |
|----------|---------|
| `GET /v2/messages/{sourceDomainId}?transactionHash={hash}` | Get attestation |
| `GET /v2/publicKeys` | Get Circle's public keys |
| `POST /v2/reattest/{nonce}` | Re-attest soft finality messages |
| `GET /v2/fastBurn/USDC/allowance` | Fast Transfer capacity |
| `GET /v2/burn/USDC/fees/{src}/{dst}` | Fee information |

**Rate Limit:** 35 requests/second; exceeding triggers 5-minute block (HTTP 429).

**API Hosts:**
- Testnet: `https://iris-api-sandbox.circle.com`
- Mainnet: `https://iris-api.circle.com`

**Verification Process:**
1. Get Circle's public keys from `/v2/publicKeys`
2. Hash message with keccak256
3. Parse 65-byte attestation signatures (r=32, s=32, v=1 bytes each)
4. Recover signer addresses via ECDSA
5. Compare against public keys (threshold validation)

### 1.13 Migration from V1 to V2

**Timeline:** 10-month deprecation starting July 2026.

**Breaking Changes:**
- All V2 contracts deployed at different addresses
- `depositForBurn` gains 3 new parameters: `destinationCaller`, `maxFee`, `minFinalityThreshold`
- `depositForBurnWithCaller` removed (use `destinationCaller` parameter instead)
- `replaceDepositForBurn` removed (no V2 equivalent)
- New: `depositForBurnWithHook`, `getMinFeeAmount`

**API Changes:**
| V1 Endpoint | V2 Replacement |
|---|---|
| `GET /v1/attestations/{messageHash}` | `GET /v2/messages/{sourceDomainId}?transactionHash={hash}` |
| `GET /v1/messages/{srcDomain}/{txHash}` | Same V2 endpoint above |

V2 eliminates message extraction from transaction receipts -- API returns pre-parsed decoded data.

### 1.14 Quickstarts

**EVM-to-EVM (Ethereum Sepolia to Arc Testnet):**
```typescript
import { createPublicClient, createWalletClient, http } from 'viem';

// Step 1: Approve USDC
await walletClient.writeContract({
  address: USDC_ADDRESS,
  abi: ['function approve(address,uint256)'],
  functionName: 'approve',
  args: [TOKEN_MESSENGER, amount]
});

// Step 2: Burn (depositForBurn)
const burnTx = await walletClient.writeContract({
  address: TOKEN_MESSENGER,
  abi: tokenMessengerAbi,
  functionName: 'depositForBurn',
  args: [amount, destDomain, mintRecipient, burnToken, destinationCaller, maxFee, 1000n]
});

// Step 3: Poll for attestation
const attestation = await pollAttestation(sourceDomain, burnTxHash);

// Step 4: Mint on destination
await destWalletClient.writeContract({
  address: MESSAGE_TRANSMITTER,
  abi: ['function receiveMessage(bytes,bytes)'],
  functionName: 'receiveMessage',
  args: [message, attestation]
});
```

**Solana-to-EVM:** Similar flow using `@solana/kit` for burn on Solana + viem for mint on EVM. Requires deriving multiple PDAs and using the 8-byte instruction discriminator from SHA-256 of `global:deposit_for_burn`.

### 1.15 Troubleshooting

**Common Issues:**
- **Stuck attestation**: 404 is normal during processing. Poll every 5 seconds, max 20 minutes.
- **Failed mint**: CCTP minting is idempotent (unique nonce). Retry safely with `receiveMessage`.
- **Nonce already used**: USDC already received -- check balance.
- **Wrong contract address**: Verify `MessageTransmitterV2` for chain.
- **Missing Solana token account**: Create USDC ATA before retry.
- **Expired attestation**: Get fresh attestation via `/v2/reattest`.
- **Destination caller mismatch**: Only the specified `destinationCaller` can mint.

### 1.16 Events

| Event | Key Parameters |
|-------|---------------|
| `DepositForBurn` | nonce (indexed), burnToken (indexed), amount, depositor (indexed), minFinalityThreshold |
| `MessageSent` | message (bytes) |
| `MessageReceived` | caller (indexed), sourceDomain (indexed), nonce (indexed), messageBody |
| `MintAndWithdraw` | mintRecipient (indexed), amount, mintToken (indexed) |

### 1.17 Security Audits

CCTP V2 contracts audited by ChainSecurity and OtterSec.

---

## 2. CCTP v1 (Legacy)

### 2.1 Overview

CCTP V1 supports **Standard Transfer only** (hard finality required, ~13-19 min for Ethereum/L2s). Being deprecated over 10 months starting July 2026.

### 2.2 Supported Chains (V1)

11 mainnet chains: Aptos, Arbitrum, Avalanche, Base, Ethereum, Noble, OP Mainnet, Polygon PoS, Solana, Sui, Unichain.

**V1 Domain Mapping:**

| Domain | Chain | Domain | Chain |
|--------|-------|--------|-------|
| 0 | Ethereum | 6 | Base |
| 1 | Avalanche | 7 | Polygon PoS |
| 2 | OP | 8 | Sui |
| 3 | Arbitrum | 9 | Aptos |
| 4 | Noble | 10 | Unichain |
| 5 | Solana | | |

### 2.3 V1 Contract Addresses (EVM Mainnet)

Contracts differ per chain (unlike V2). Example:
- **Ethereum TokenMessenger**: `0xBd3fa81B58Ba92a82136038B25aDec7066af3155`
- **Avalanche TokenMessenger**: `0x6B25532e1060CE10cc3B0A99e5683b91BFDe6982`
- **Base TokenMessenger**: `0x1682Ae6375C4E4A97e4B583BC394c861A46D8962`

### 2.4 V1 Message Format

Smaller header (116 bytes):
- nonce is uint64 (8 bytes) vs V2's bytes32 (32 bytes)
- No `minFinalityThreshold` or `finalityThresholdExecuted` fields
- BurnMessage body lacks `maxFee`, `feeExecuted`, `expirationBlock`, `hookData`

### 2.5 V1 Block Confirmations

| Chain | Confirmations | Time |
|-------|---------------|------|
| Ethereum | ~65 | 13-19 min |
| Avalanche | 1 | ~8 sec |
| Solana | 32 | ~25 sec |
| Polygon PoS | ~33 | 75-120 sec |
| Sui, Aptos | 1 | ~8 sec |
| Noble | 1 | ~20 sec |
| Arbitrum, Base, OP, Unichain | ~65 ETH blocks | 13-19 min |

### 2.6 V1 APIs

- `GET /v1/attestations/{messageHash}` -- Get signed attestation
- `GET /v1/messages/{sourceDomainId}/{transactionHash}` -- Transaction details
- `GET /v1/publicKeys` -- Circle's public keys

### 2.7 Non-EVM Implementations

**Noble (Cosmos):** CCTP operates as a native Cosmos SDK module (domain 4). Module address: `noble12l2w4ugfz4m6dd73yysz477jszqnfughxvkss5`. Supports IBC composable flows.

**Solana (V1):**
- MessageTransmitter: `CCTPmbSD7gX1bxKPAmg77w8oFzNFpaQiQUWD43TKaecd`
- TokenMessengerMinter: `CCTPiPYPc6AsJuwueEnWgSgucamXDZwBd53dQ11YiKX3`

**Aptos:**
- Mainnet MessageTransmitter: `0x177e17751820e4b4371873ca8c30279be63bdea63b88ed0f2239c2eea10f1772`
- Mainnet TokenMessengerMinter: `0x9bce6734f7b63e835108e3bd8c36743d4709fe435f44791918801d0989640a9d`
- Uses Hot Potato Receipt pattern for atomic message handling

**Sui:**
- Mainnet MessageTransmitter: `0x08d87d37ba49e785dde270a83f8e979605b03dc552b5548f26fdf2f49bf7ed1b`
- Mainnet TokenMessengerMinter: `0x2aa6c5d56376c371f88a6cc42e852824994993cb9bab8d3e6450cbe3cb32b94e`
- Uses Ticket Pattern for upgradeable composability and Auth Struct for package identity verification

### 2.8 V1 Limits

- **Per-message burn limit**: Configurable per chain, managed by Circle
- **Minter allowance**: Per-minter limit decremented on each mint

### 2.9 Key V1 Limitation

In V1 it is NOT possible to perform burn-and-mint for USDC and include arbitrary data in the same message. V2 solves this with hooks.

---

## 3. Bridge Kit SDK

### 3.1 Overview

Bridge Kit is a high-level SDK built on top of CCTP that simplifies cross-chain USDC transfers to a single method call. Client-side and server-side compatible.

**NPM Packages:**
- `@circle-fin/bridge-kit` -- Core SDK
- `@circle-fin/adapter-viem-v2` -- Viem adapter for EVM
- `@circle-fin/adapter-ethers-v6` -- Ethers adapter for EVM
- `@circle-fin/adapter-solana-kit` -- Solana adapter
- `@circle-fin/adapter-circle-wallets` -- Circle Wallets adapter (server-only)

### 3.2 Installation

**EVM Only (Viem):**
```bash
npm install @circle-fin/bridge-kit @circle-fin/adapter-viem-v2 viem
```

**EVM + Solana:**
```bash
npm install @circle-fin/bridge-kit @circle-fin/adapter-viem-v2 @circle-fin/adapter-solana-kit @solana/kit @solana/web3.js viem
```

**Circle Wallets (server-only):**
```bash
npm install @circle-fin/bridge-kit @circle-fin/adapter-circle-wallets
```

Choose Viem OR Ethers (not both) for EVM.

### 3.3 Adapter Setup

**Standard (Private Key):**
```typescript
const adapter = createViemAdapterFromPrivateKey({
  privateKey: process.env.PRIVATE_KEY as string,
});
```

**Custom RPC:**
```typescript
const adapter = createViemAdapterFromPrivateKey({
  privateKey: process.env.PRIVATE_KEY,
  rpcByChainName: { Ethereum: 'https://eth-mainnet.alchemyapi.io/v2/...' }
});
```

**Browser Wallet:**
```typescript
const adapter = createViemAdapterFromProvider({ provider: window.ethereum });
```

**Circle Wallets:**
```typescript
const adapter = createCircleWalletsAdapter({
  apiKey: process.env.CIRCLE_API_KEY,
  entitySecret: process.env.CIRCLE_ENTITY_SECRET,
});
```

**Solana:**
```typescript
const adapter = createSolanaKitAdapterFromPrivateKey({
  privateKey: process.env.SOLANA_PRIVATE_KEY, // Base58, Base64, or JSON array
});
```

### 3.4 Core API

**Bridge (Execute Transfer):**
```typescript
const kit = new BridgeKit();
const result = await kit.bridge({
  from: { adapter, chain: "Base_Sepolia" },
  to: { adapter, chain: "Arc_Testnet" },
  amount: "1.00",
  config: {
    transferSpeed: "FAST",  // "FAST" (default) or "SLOW"
    maxFee: "5",            // max fee in USDC
    customFee: { value: "1.50", recipientAddress: "0x..." }
  }
});
```

**Estimate Costs:**
```typescript
const estimate = await kit.estimate({
  from: { adapter, chain: "HyperEVM_Testnet" },
  to: { adapter, chain: "Arc_Testnet" },
  amount: "10.00"
});
// Returns: gasFees[], fees[] (with "kit" and "provider" types)
```

**Retry Failed Transfer:**
```typescript
const retryResult = await kit.retry(failedResult, { from: { adapter }, to: { adapter } });
```

**Event Handling:**
```typescript
kit.on("approve", (payload) => console.log(payload.values.txHash));
kit.on("burn", (payload) => console.log(payload.values.txHash));
kit.on("fetchAttestation", (payload) => console.log(payload.values.data.attestation));
kit.on("mint", (payload) => console.log(payload.values.txHash));
kit.on("*", (payload) => { /* wildcard listener */ });
```

### 3.5 Transfer Process (4 Steps)

1. **approve** -- Authorize contract to spend USDC
2. **burn** -- Burn USDC on source chain, generate attestation request
3. **fetchAttestation** -- Obtain Circle's cryptographic signature
4. **mint** -- Create USDC on destination chain

### 3.6 Custom Fees

Circle keeps 10% of custom fees; 90% goes to fee recipient on source chain.

**Per-Call Fee:**
```typescript
config: { customFee: { value: "1.50", recipientAddress: "0x..." } }
```

**Global Fee Policy:**
```typescript
kit.setCustomFeePolicy({
  computeFee: (params) => params.source.chain.chain === Blockchain.Arc_Testnet ? "1.00" : "1.50",
  resolveFeeRecipientAddress: (chain) => chain.chain === Blockchain.Arc_Testnet ? "0xA..." : "0xB...",
});
```

### 3.7 Forwarding Service

```typescript
const result = await kit.bridge({
  from: { adapter, chain: "Ethereum" },
  to: { chain: "Base", recipientAddress: "0x...", useForwarder: true },
  amount: "100.50",
});
```
No destination adapter needed when using forwarding with `recipientAddress`.

### 3.8 Recipient Address

```typescript
to: { adapter, chain: "Base_Sepolia", recipientAddress: "0x1234..." }
```
Default: tokens go to same address as signing wallet.

### 3.9 Transfer Speed Configuration

```typescript
// Slow (Standard Transfer)
config: { transferSpeed: "SLOW" }

// Fast with max fee cap (auto-downgrades to SLOW if exceeded)
config: { transferSpeed: "FAST", maxFee: "5" }
```

### 3.10 Error Handling

**Error Types:**
| Type | Code Range | Description |
|------|-----------|-------------|
| INPUT | 1000-1999 | Validation failures |
| NETWORK | 3000-3999 | Connectivity problems |
| RPC | 4000-4999 | Provider issues |
| ONCHAIN | 5000-5999 | Transaction failures |
| BALANCE | 9000-9999 | Insufficient funds |

**Recoverability:** FATAL, RETRYABLE, RESUMABLE

**Type Guards:** `isKitError()`, `isFatalError()`, `isRetryableError()`, `isInputError()`, `isBalanceError()`, `isOnchainError()`, `isRpcError()`, `isNetworkError()`

### 3.11 Supported Chains

**Mainnet (18 chains):** Arbitrum, Avalanche, Base, Codex, Ethereum, HyperEVM, Ink, Linea, Monad, OP Mainnet, Plume, Polygon PoS, Sei, Solana, Sonic, Unichain, World Chain, XDC

**Circle Wallets Adapter** supports a subset (10 chains): Arbitrum, Avalanche, Base, Ethereum, Monad, OP Mainnet, Polygon PoS, Solana, Unichain + testnets.

### 3.12 BridgeResult Structure

```typescript
interface BridgeResult {
  state: "pending" | "success" | "error";
  amount: string;
  token: string;
  provider: string;
  source: { chain: ChainDefinition; address: string };
  destination: { chain: ChainDefinition; address: string };
  steps: BridgeStep[];
}

interface BridgeStep {
  name: string;  // "approve", "burn", "fetchAttestation", "mint"
  state: "pending" | "success" | "error" | "noop";
  txHash?: string;
  explorerUrl?: string;
  errorMessage?: string;
}
```

---

## 4. Gateway

### 4.1 Overview

Gateway provides a **unified USDC balance** across multiple blockchains. Users deposit USDC into non-custodial smart contracts on source chains, then mint equivalent amounts on destination chains via API calls -- all within **under 500 milliseconds**.

**Key Distinction from CCTP:** Gateway front-loads the finalization wait time rather than requiring finality mid-transfer. Users pre-deposit, then access funds instantly.

**Non-Custodial:** 7-day trustless withdrawal option for API outage scenarios.

### 4.2 Architecture

**Smart Contracts:**
- **GatewayWallet**: Accepts deposits, manages balances, handles withdrawals and delegation
- **GatewayMinter**: Accepts signed attestations, mints USDC on destination chains

**Offchain System:** Ledger tracking per-chain/per-token/per-address balances.

### 4.3 Contract Addresses

**Testnet (EVM):**
- GatewayWallet: `0x0077777d7EBA4688BDeF3E311b846F25870A19B9` (all chains)
- GatewayMinter: `0x0022222ABE238Cc2C7Bb1f21003F0a260052475B` (all chains)

**Testnet (Solana):**
- GatewayWallet: `GATEwdfmYNELfp5wDmmR6noSr2vHnAfBPMm2PvCzX5vu`
- GatewayMinter: `GATEmKK2ECL1brEngQZWCgMWPbvrEYqsV6u29dAaHavr`

**Mainnet (EVM):**
- GatewayWallet: `0x77777777Dcc4d5A8B6E418Fd04D8997ef11000eE` (all chains)
- GatewayMinter: `0x2222222d7164433c4C09B0b0D809a9b52C04C205` (all chains)

**Mainnet (Solana):**
- GatewayWallet: `GATEwy4YxeiEbRJLwB6dXgg7q61e6zBPrMzYj5h1pRXQ`
- GatewayMinter: `GATEm5SoBJiSw1v2Pz1iPBgUYkXzCUJ27XSXhDfSyzVZ`

### 4.4 Supported Chains

**Testnet (13 chains):** Arc, Arbitrum Sepolia, Avalanche Fuji, Base Sepolia, Ethereum Sepolia, HyperEVM Testnet, OP Sepolia, Polygon Amoy, Sei Atlantic, Solana Devnet, Sonic Testnet, Unichain Sepolia, World Chain Sepolia.

**Mainnet (12 chains):** Arbitrum, Avalanche, Base, Ethereum, HyperEVM, OP, Polygon PoS, Sei, Solana, Sonic, Unichain, World Chain.

**Confirmation Times:**
- Arc Testnet: ~0.5 seconds
- Avalanche, Polygon, Solana, Sonic: ~8 seconds
- HyperEVM, Sei: ~5 seconds
- Ethereum, Arbitrum, Base, OP, Unichain, World Chain: ~13-19 minutes (~65 ETH blocks)

### 4.5 Deposit Flow

**EVM:**
```typescript
// Step 1: Approve
await walletClient.writeContract({
  address: USDC_ADDRESS, abi: erc20Abi,
  functionName: 'approve',
  args: [GATEWAY_WALLET, amount]
});

// Step 2: Deposit
await walletClient.writeContract({
  address: GATEWAY_WALLET, abi: gatewayWalletAbi,
  functionName: 'deposit',
  args: [USDC_ADDRESS, amount]
});
```

**WARNING:** Directly transferring USDC to the Gateway Wallet with standard ERC-20 transfer will result in LOSS of funds.

**Solana:** Uses Anchor framework with PDA derivation for wallet, custody, deposit, and denylist accounts.

### 4.6 Transfer Flow (Instant)

1. **Create Burn Intents**: Define TransferSpec + BurnIntent with EIP-712 typed data
2. **Sign**: Account holder signs with private key
3. **Submit to API**: `POST https://gateway-api-testnet.circle.com/v1/transfer`
4. **Mint**: Use returned attestation + signature to call `gatewayMint` on destination GatewayMinter

**Key Data Structures:**

```typescript
// TransferSpec
{
  version, sourceDomain, destinationDomain,
  sourceContract, destinationContract,
  sourceToken, destinationToken,
  depositor, recipient, signer, caller,
  value, salt, hookData
}

// BurnIntent
{
  maxBlockHeight,  // Expiration on source chain
  maxFee,          // Max fee allowance
  spec: TransferSpec
}
```

**BurnIntentSet:** Up to 16 burn intents with single EIP-712 signature on EVM. NOT supported on Solana.

### 4.7 Fees

**Transfer Fee:** 0.005% (0.5 basis points) -- crosschain only, same-chain = free

**Gas Fees (per burn intent):**

| Chain | Gas Fee |
|-------|---------|
| Sei, Unichain | $0.001 |
| OP, Polygon PoS | $0.0015 |
| Arbitrum, Base, Sonic, World Chain | $0.01 |
| Avalanche | $0.02 |
| HyperEVM | $0.05 |
| Solana | $0.15 |
| Ethereum | $2.00 |

**Formula:** `maxFee >= gas fee + (transfer amount * 0.00005)`

### 4.8 Gateway Forwarding Service

Automates destination chain minting for Gateway transfers.

**Enable:** Add `enableForwarder=true` to `/v1/transfer` and `/v1/estimate` endpoints.

**Forwarding Fees:**
- Ethereum: $1.25 per transfer
- Other chains: $0.20 per transfer

**Formula:** `maxFee >= gas fee + forwarding fee + (transfer amount * 0.00005)`

**Limitation:** Solana not supported as destination chain for forwarding.

### 4.9 Contract Interfaces

**GatewayWallet Functions:**
- `deposit(token, value)` -- Direct deposit
- `depositFor(token, depositor, value)` -- Deposit on behalf of another
- `depositWithPermit(...)` -- EIP-2612 permit deposit
- `depositWithAuthorization(...)` -- ERC-3009 authorization deposit
- `addDelegate(token, delegate)` / `removeDelegate(token, delegate)` -- SCA delegation
- `initiateWithdrawal(token, value)` / `withdraw(token)` -- Two-phase trustless withdrawal (7-day delay)
- Balance queries: `totalBalance()`, `availableBalance()`, `withdrawingBalance()`, `withdrawableBalance()`

**GatewayMinter Functions:**
- `gatewayMint(attestationPayload, signature)` -- Mint via signed attestation

**Key Events:**
- `Deposited` -- Token deposit logged
- `GatewayBurned` -- Cross-chain burn with destination domain and fee info
- `AttestationUsed` -- Minting with source domain and transferSpecHash
- `WithdrawalInitiated` / `WithdrawalCompleted` -- Withdrawal lifecycle

### 4.10 Delegate Management (for Smart Contract Accounts)

SCAs cannot sign EIP-712 burn intents directly. Solution: authorize an EOA as delegate.

```typescript
// Add delegate
await walletClient.writeContract({
  address: GATEWAY_WALLET,
  functionName: 'addDelegate',
  args: [USDC_ADDRESS, delegateEOAAddress]
});

// Remove delegate (existing signed burn intents remain valid)
await walletClient.writeContract({
  address: GATEWAY_WALLET,
  functionName: 'removeDelegate',
  args: [USDC_ADDRESS, delegateEOAAddress]
});
```

### 4.11 Solana-Specific Details

- **BurnIntentSets NOT supported** -- each Solana transfer must be a separate signed BurnIntent
- Signing uses Ed25519 with 16-byte prefix (`0xff000000000000000000000000000000`)
- Uses `ReducedMintAttestation` format to fit Solana's 1232-byte transaction limit
- Message encoding uses custom buffer layouts with magic numbers:
  - TransferSpec: `0xca85def7`
  - BurnIntent: `0x070afbc2`
- `maxBlockHeight` refers to Solana slot, not traditional block height
- Attestation buffers can span multiple transactions for large payloads (max 10KB)

### 4.12 API Endpoints

| Endpoint | Purpose |
|----------|---------|
| `POST /v1/balances` | Get unified balance across chains |
| `POST /v1/deposits` | Get pending deposit status |
| `POST /v1/transfer` | Submit burn intents, receive attestation |
| `POST /v1/estimate` | Estimate transfer fees |
| `GET /v1/transfer/{id}` | Check transfer status |

**API Hosts:**
- Testnet: `https://gateway-api-testnet.circle.com`
- Mainnet: `https://gateway-api.circle.com`

### 4.13 Special Chain Considerations

- **Arbitrum:** `maxBlockHeight` uses Ethereum L1 block height, not L2
- **Solana:** Distinct encoding, signing protocols; no BurnIntentSets
- **All chains:** TransferSpecHash serves dual purpose -- cross-chain identifier AND replay protection

### 4.14 Security

- Smart contracts audited by ChainSecurity and OtterSec
- Non-custodial with 7-day trustless withdrawal
- Denylist mechanism for compliance

---

## 5. Nanopayments & x402 Protocol

### 5.1 Nanopayments Overview

Gateway Nanopayments enables gas-free USDC payments as small as **$0.000001** by batching thousands of transactions into single onchain settlements.

**Key Innovation:** Buyers sign offchain EIP-3009 `TransferWithAuthorization` messages (zero gas), sellers verify and serve immediately, Gateway settles net positions in bulk.

### 5.2 Payment Flow

1. Buyer deposits USDC into Gateway Wallet (one-time onchain tx)
2. Buyer requests paid resource from seller
3. Seller responds with HTTP 402 + payment requirements
4. Buyer signs EIP-3009 authorization (offchain, zero gas)
5. Buyer retries with signed authorization
6. Seller verifies signature, serves resource immediately
7. Gateway batches authorizations and settles net positions

### 5.3 x402 Protocol

An open standard for internet-native payments built on HTTP 402 Payment Required.

**Three Headers:**

| Header | Direction | Function |
|--------|-----------|----------|
| `PAYMENT-REQUIRED` | Server -> Client | Payment terms |
| `PAYMENT-SIGNATURE` | Client -> Server | Signed payment authorization |
| `PAYMENT-RESPONSE` | Server -> Client | Payment confirmation |

**Protocol Flow:**
1. Client GETs protected resource
2. Server returns 402 with accepted payment schemes
3. Client signs payment, retries with `PAYMENT-SIGNATURE` header
4. Server verifies, returns resource + `PAYMENT-RESPONSE` header

### 5.4 Batched Settlement Architecture

**Three-Component Security Model:**
- **TEE (AWS Nitro Enclave)**: Verifies signatures, computes net changes, signs batch with KMS-protected keys
- **Onchain Verification**: Smart contract validates TEE signature before executing batch
- **Cryptographic Attestations**: Proves enclave runs specific audited code

**Balance States:**
- `available`: Spendable balance
- When authorization created: funds locked (removed from available)
- After settlement: net positions applied

### 5.5 EIP-3009 Signing for Nanopayments

**Domain:**
```typescript
{
  name: "GatewayWalletBatched",
  version: "1",
  chainId: 5042002,  // Arc Testnet
  verifyingContract: GATEWAY_WALLET_ADDRESS
}
```

**Typed Data:**
```typescript
const types = {
  TransferWithAuthorization: [
    { name: "from", type: "address" },
    { name: "to", type: "address" },
    { name: "value", type: "uint256" },
    { name: "validAfter", type: "uint256" },
    { name: "validBefore", type: "uint256" },
    { name: "nonce", type: "bytes32" }
  ]
};
```

**Critical:** `validBefore` must be at least 3 days in the future.

### 5.6 Buyer SDK Implementation

```typescript
import { GatewayClient } from "@circlefin/x402-batching/client";

const client = new GatewayClient({
  chain: "arcTestnet",
  privateKey: process.env.PRIVATE_KEY as `0x${string}`,
});

// Deposit (one-time)
await client.deposit(1_000_000n);  // 1 USDC

// Pay for resource
const { data, status } = await client.pay("https://api.example.com/premium-data");

// Check balance
const balances = await client.getBalances();

// Withdraw
await client.withdraw(500_000n);  // 0.5 USDC
```

**Key Methods:**
- `deposit(amount)` -- Deposit USDC to Gateway
- `pay<T>(url)` -- Full x402 negotiation automatically
- `withdraw(amount, options?)` -- Same-chain or cross-chain withdrawal
- `getBalances()` -- Check wallet + Gateway balances
- `supports(url)` -- Check x402 support

### 5.7 Seller SDK Implementation

```typescript
import express from "express";
import { createGatewayMiddleware } from "@circlefin/x402-batching/server";

const app = express();
const gateway = createGatewayMiddleware({
  sellerAddress: "0xYOUR_WALLET_ADDRESS",
  networks: ["eip155:5042002"],  // Optional: restrict chains
});

app.get("/premium-data", gateway.require("$0.01"), (req, res) => {
  const { payer, amount, network } = req.payment!;
  res.json({ secret: "Premium content", paid_by: payer });
});

app.listen(3000);
```

**For non-Express:**
```typescript
import { BatchFacilitatorClient } from "@circlefin/x402-batching/server";
const facilitator = new BatchFacilitatorClient();
const settlement = await facilitator.settle(payload, requirements);
```

### 5.8 Nanopayments SDK Reference

**Package:** `@circle-fin/x402-batching`

**Buyer Exports (`/client`):**
- `GatewayClient` -- Full client for deposits, payments, withdrawals
- `BatchEvmScheme` -- Low-level payment scheme implementation
- `registerBatchScheme` -- Register with x402Client

**Seller Exports (`/server`):**
- `createGatewayMiddleware` -- Express middleware factory
- `BatchFacilitatorClient` -- Direct settlement client
  - `verify(payload, requirements)` -- Validate payment
  - `settle(payload, requirements)` -- Batch for settlement
  - `getSupported()` -- List supported schemes/networks

**Constants:**
- `CIRCLE_BATCHING_NAME`: `'GatewayWalletBatched'`
- `CIRCLE_BATCHING_VERSION`: `'1'`
- `CIRCLE_BATCHING_SCHEME`: `'exact'`

---

## 6. Release Notes (2026)

### 6.1 Bridge Kit 2026

| Date | Change |
|------|--------|
| 2026.02.27 | Forwarding Service support added |
| 2026.01.28 | Monad mainnet + testnet support |
| 2026.01.14 | Enhanced `estimate()` with token/amount/source/destination fields; error type guards (`isKitError`, `isBalanceError`, `isOnchainError`); helper functions (`getErrorCode`, `getErrorMessage`) |

### 6.2 CCTP 2026

| Date | Change |
|------|--------|
| 2026.02.26 | Morph Hoodi Testnet support |
| 2026.02.XX | HyperCore transfer documentation modernized |
| 2026.02.09 | EDGE Testnet support |
| 2026.01.22 | Circle Forwarding Service launched |
| 2026.01.12 | Complete documentation rewrite; Bridge Kit emphasized as preferred integration |
| 2026.01.08 | Finality-indexed pricing model |

### 6.3 Gateway 2026

| Date | Change |
|------|--------|
| 2026.02.11 | Gateway Forwarding Service reference added |
| 2026.02.07 | SCA delegate management support |
| 2026.02.02 | Documentation standardization |
| 2026.01.30 | `POST /v1/deposits` endpoint added |
| 2026.01.14 | Solana support launched with full quickstarts |

---

## Quick Reference: Product Comparison

| Feature | CCTP v2 | Bridge Kit | Gateway |
|---------|---------|-----------|---------|
| **Speed** | 8-20s (Fast), 15-19min (Std) | Same as CCTP | <500ms |
| **Model** | Point-to-point burn/mint | Wrapper around CCTP | Unified balance |
| **Fees** | 0-14 bps (Fast), Free (Std) | Same + custom fees | 0.5 bps + gas |
| **Chains** | 23+ | 18+ | 12+ |
| **Complexity** | Manual contract calls | Single SDK call | Deposit + API + mint |
| **Best For** | Direct integration | Quick integration | Instant transfers |
| **Custody** | Self-managed | Self-managed | Non-custodial (7-day trustless) |
| **Hooks** | Yes (V2) | Via CCTP | hookData field |
| **Forwarding** | Yes (limited chains) | Yes | Yes (not Solana dest) |
