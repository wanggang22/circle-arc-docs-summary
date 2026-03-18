# Arc Network Documentation - Complete Technical Reference

> Source: https://docs.arc.network (24 pages)
> Generated: 2026-03-04
> Coverage: Concepts (7) | References (5) | Tools (5) | Tutorials (8)

---

## Table of Contents

- [Part 1: Concepts](#part-1-concepts)
  - [1.1 Welcome to Arc](#11-welcome-to-arc)
  - [1.2 System Overview](#12-system-overview)
  - [1.3 Consensus Layer](#13-consensus-layer)
  - [1.4 Execution Layer](#14-execution-layer)
  - [1.5 Deterministic Finality](#15-deterministic-finality)
  - [1.6 Opt-in Privacy](#16-opt-in-privacy)
  - [1.7 Stable Fee Design](#17-stable-fee-design)
- [Part 2: References](#part-2-references)
  - [2.1 Connect to Arc](#21-connect-to-arc)
  - [2.2 Contract Addresses](#22-contract-addresses)
  - [2.3 EVM Compatibility](#23-evm-compatibility)
  - [2.4 Gas and Fees](#24-gas-and-fees)
  - [2.5 Sample Applications](#25-sample-applications)
- [Part 3: Tools](#part-3-tools)
  - [3.1 Account Abstraction](#31-account-abstraction)
  - [3.2 Block Explorers](#32-block-explorers)
  - [3.3 Compliance Vendors](#33-compliance-vendors)
  - [3.4 Data Indexers](#34-data-indexers)
  - [3.5 Node Providers](#35-node-providers)
- [Part 4: Tutorials](#part-4-tutorials)
  - [4.1 Deploy on Arc (Foundry)](#41-deploy-on-arc-foundry)
  - [4.2 Deploy Contracts (Circle SDK)](#42-deploy-contracts-circle-sdk)
  - [4.3 Interact with Contracts](#43-interact-with-contracts)
  - [4.4 Monitor Contract Events](#44-monitor-contract-events)
  - [4.5 Bridge USDC to Arc](#45-bridge-usdc-to-arc)
  - [4.6 Transfer USDC or EURC](#46-transfer-usdc-or-eurc)
  - [4.7 Access USDC Crosschain (Gateway)](#47-access-usdc-crosschain-gateway)
  - [4.8 Register Your First AI Agent (ERC-8004)](#48-register-your-first-ai-agent-erc-8004)

---

# Part 1: Concepts

## 1.1 Welcome to Arc

Arc is an **open Layer-1 blockchain** purpose-built to unite programmable money and onchain innovation with real-world economic activity.

### Core Properties

| Property | Detail |
|----------|--------|
| Type | Public EVM-compatible L1 |
| Consensus | Malachite (BFT engine) |
| Finality | Deterministic, sub-second |
| Gas Token | USDC (stablecoin) |
| Validator Model | Permissioned (institutional) |
| Network Access | Fully open / permissionless |

### Key Features
- **Fiat-based fees** - Gas denominated in USDC, not volatile tokens
- **Deterministic sub-second finality** - No probabilistic confirmation
- **Opt-in configurable privacy** - Roadmap feature for confidential transfers
- **Circle full-stack integration** - CCTP, Gateway, Developer Wallets, Smart Contract Platform

### Target Use Cases
1. Onchain credit
2. Capital markets settlement
3. Stablecoin FX
4. Agentic commerce
5. Cross-border payments

### Multichain Alignment
- Circle CCTP for cross-chain USDC transfers
- Circle Gateway for chain-abstracted balances
- Standard EVM tooling (Foundry, Hardhat, ethers.js, etc.)

---

## 1.2 System Overview

Arc's architecture consists of two fundamental layers working in tandem:

### Architecture Diagram

```
                    +---------------------------+
                    |     Consensus Layer        |
                    |       (Malachite)          |
                    | - BFT protocol             |
                    | - Transaction ordering     |
                    | - Block finalization        |
                    | - Proof-of-Authority       |
                    +---------------------------+
                              |
                    (ordered transactions)
                              |
                    +---------------------------+
                    |     Execution Layer        |
                    |        (Reth)              |
                    | - Ledger & State           |
                    | - Fee Manager (USDC)       |
                    | - Privacy Module           |
                    | - Stablecoin Services      |
                    +---------------------------+
```

### Consensus Layer (Malachite)
- High-performance implementation of **Tendermint BFT** protocol
- Deterministic finality with sub-one-second block finalization
- Irreversible - no transaction reorganization or rollback
- Proof-of-Authority validator model

### Execution Layer (Reth)
- Rust-based Ethereum execution client
- Maintains blockchain ledger, state, accounts, balances, smart contracts
- Modular components:
  - **Ledger & State** - Accounts, balances, contracts, transaction records
  - **Fee Manager** - Stabilizes fees using USDC as accounting unit
  - **Privacy Module** - Confidential transfers and selective disclosure (roadmap)
  - **Stablecoin Services** - Multi-currency payments, FX conversions, settlement (roadmap)

### Data Flow
1. Consensus layer determines transaction order and block finality
2. Execution layer applies ordered transactions to the ledger
3. Processing routed through modular components (Fee Manager, Privacy, Stablecoin)

---

## 1.3 Consensus Layer

### Malachite Consensus Engine

Arc uses **Malachite**, a high-performance Tendermint BFT protocol variant with Proof-of-Authority validation.

### Performance Specifications

| Metric | Value |
|--------|-------|
| Finality Type | Deterministic (irreversible) |
| Finality Speed | < 1 second (normal conditions) |
| Latency | < 350ms (benchmark) |
| Throughput (20 validators) | 3,000+ TPS |
| Throughput (reduced set) | > 10,000 TPS |

### Four-Phase Consensus Process

```
1. PROPOSE    --> Designated validator creates and broadcasts block
2. PRE-VOTE   --> Validators broadcast validity assessments
3. PRE-COMMIT --> Second voting round
4. COMMIT     --> >2/3 pre-commitment = block finalized locally across all validators
```

### Validator Architecture

| Property | Detail |
|----------|--------|
| Selection Model | Permissioned institutions with compliance obligations |
| Distribution | Geographic diversity across multiple global regions |
| Block Rotation | Fair distribution of proposal responsibility |
| Accountability | Real-world institutional consequences for malicious behavior |

### Security Parameters

| Parameter | Value |
|-----------|-------|
| Safety Threshold | < 1/3 faulty validators = no conflicting blocks finalized |
| Liveness Requirement | >= 2/3 validators online and honest |
| Reorganization Resistance | Protocol prevents block reorgs by design |

### Roadmap Enhancements
- Multi-proposer support (concurrent block proposals)
- Latency reduction via 3-to-2 round optimization
- Potential evolution toward permissioned Proof-of-Stake

---

## 1.4 Execution Layer

### Reth (Rust Ethereum Execution Client)

Arc's execution layer is built on **Reth**, a Rust implementation of the Ethereum execution client providing performance, safety, and modularity.

### Three Core Responsibilities

**1. Ledger Maintenance**
- Tracks accounts, balances, and smart contracts
- Stores contract code, state variables, transaction records

**2. Transaction Execution**
- Applies EVM logic for smart contracts and transfers
- Deducts gas via the Fee Manager
- Invokes Arc modules (Privacy, Stablecoin Services) as needed

**3. Validation**
- Ensures transaction validity before consensus finalization
- Rejects invalid transactions
- Produces a state root hash which consensus finalizes

### Processing Pipeline

```
Transaction Pool (pending txs)
    --> Sequential Block Execution (updating state)
        --> Gas Accounting (Fee Manager)
            --> Module Routing (Privacy, Stablecoin Services)
                --> Merkle Root State Hash Generation
```

### Arc-Specific Extensions

| Extension | Status | Description |
|-----------|--------|-------------|
| Fee Manager | Active | Stabilizes fees using USDC as accounting unit |
| Privacy Module | Roadmap | Confidential transfers with selective disclosure |
| Stablecoin Services | Roadmap | Cross-currency settlement, multi-stablecoin payments |

---

## 1.5 Deterministic Finality

### Finality Model

Arc implements **deterministic** (not probabilistic) finality. Transactions achieve exactly one of two states:
- **Unconfirmed** - Not yet committed
- **Irreversibly Final** - Committed and permanent

There is NO intermediate "probably final" state.

### Specifications

| Property | Value |
|----------|-------|
| Block Finality | < 1 second |
| Transaction Settlement | Sub-second |
| Reorg Risk | Zero once finalized |
| Confirmation Count Needed | 1 (single block) |

### How It Works
- Malachite BFT consensus engine provides deterministic guarantees
- Once a block is committed, **any transaction in that block is instantly and irreversibly final**
- No chain reorganizations can reverse committed transactions

### Comparison with Other Chains

| Chain Type | Finality Model | Wait Time |
|------------|---------------|-----------|
| Proof-of-Work (Bitcoin) | Probabilistic | ~60 min (6 confirmations) |
| Proof-of-Stake (Ethereum) | Probabilistic w/ checkpoints | 12-15 minutes |
| Arc (Malachite BFT) | Deterministic | < 1 second |

### Developer Implications
- No need for retry logic, rollback handling, or reorg monitoring
- Immediate triggering of off-chain effects upon block commitment
- Simplified application architecture

---

## 1.6 Opt-in Privacy

> **Status: ON ROADMAP - Not yet available on Arc**

### Planned Phase 1: Confidential Transfers
- Transaction **amounts** encrypted and hidden from public ledger
- **Sender and receiver addresses remain visible** for analytics compatibility
- Onchain finalization maintains deterministic guarantees

### View Key System
- Grants controlled read access to encrypted transaction data
- Enables auditors, regulators, and institutions to review details when authorized
- Supports Travel Rule compliance mechanisms

### Modular Cryptographic Backend (Potential Implementations)

| Technology | Description |
|------------|-------------|
| TEE (Trusted Execution Environments) | Initial deployment target |
| MPC (Multi-Party Computation) | Distributed secret-splitting across multiple parties |
| FHE (Fully Homomorphic Encryption) | Computations on encrypted data |
| ZK (Zero-Knowledge Proofs) | Privacy-preserving verification |

### Planned Developer Capabilities
- Shield transaction amounts while maintaining participant visibility
- Grant selective disclosure permissions to compliance parties
- Design privacy-by-default applications for institutional workflows

---

## 1.7 Stable Fee Design

### Fee Formula

```
fee = gas_units * base_fee_in_USDC
```

### Key Properties

| Property | Value |
|----------|-------|
| Gas Token | USDC |
| Target Transaction Cost | ~$0.01 per transaction |
| Fee Model | Modified EIP-1559 with EWMA smoothing |
| Volatility | Eliminated (stablecoin denominated) |

### USDC as Native Gas Token
- Dollar-denominated fee estimation
- No token price volatility in gas calculations
- Transfer and payment units align (same denomination)

### Fee Smoothing Mechanism

Arc modifies Ethereum's EIP-1559 model:
- Uses **Exponentially Weighted Moving Average (EWMA)** of block utilization
- NOT per-block recalculation like standard EIP-1559
- Gradual fee adjustments (not abrupt changes)
- Reduced impact from short-term demand spikes
- Bounded base fees maintaining predictability

### Planned Stablecoin Expansion (Roadmap)
- EURC, USDT, MXNB and additional stablecoins for gas
- Paymaster-enabled fee sponsorship
- Programmatic custom fee rules

---

# Part 2: References

## 2.1 Connect to Arc

### Network Configuration (Testnet)

| Parameter | Value |
|-----------|-------|
| Chain ID | `5042002` |
| Native Gas Token | USDC |
| Token Decimals | 18 |
| Currency Symbol | USDC |

### RPC Endpoints (HTTP)

| Provider | URL |
|----------|-----|
| Primary | `https://rpc.testnet.arc.network` |
| Blockdaemon | `https://rpc.blockdaemon.testnet.arc.network` |
| dRPC | `https://rpc.drpc.testnet.arc.network` |
| QuickNode | `https://rpc.quicknode.testnet.arc.network` |

### WebSocket Endpoints

| Provider | URL |
|----------|-----|
| Primary | `wss://rpc.testnet.arc.network` |
| dRPC | `wss://rpc.drpc.testnet.arc.network` |
| QuickNode | `wss://rpc.quicknode.testnet.arc.network` |

### MetaMask Configuration

```
Network Name:    Arc Testnet
RPC URL:         https://rpc.testnet.arc.network
Chain ID:        5042002
Currency Symbol: USDC
Block Explorer:  https://testnet.arcscan.app
```

### Resources

| Resource | URL |
|----------|-----|
| Block Explorer | https://testnet.arcscan.app |
| Faucet | https://faucet.circle.com |
| Gas Tracker | https://testnet.arcscan.app/gas-tracker |

---

## 2.2 Contract Addresses

### Stablecoins

| Token | Address | Decimals | Notes |
|-------|---------|----------|-------|
| USDC (native) | `0x3600000000000000000000000000000000000000` | 18 (native), 6 (ERC-20) | Required for gas fees |
| EURC | `0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a` | 6 | Euro-denominated |

### USYC (Yield-bearing)

| Contract | Address | Decimals |
|----------|---------|----------|
| Token | `0xe9185F0c5F296Ed1797AaE4238D26CCaBEadb86C` | 6 |
| Entitlements | `0xcc205224862c7641930c87679e98999d23c26113` | - |
| Teller | `0x9fdF14c5B14173D74C08Af27AebFf39240dC105A` | - |

### CCTP (Cross-Chain Transfer Protocol) - Domain 26

| Contract | Address |
|----------|---------|
| TokenMessengerV2 | `0x8FE6B999Dc680CcFDD5Bf7EB0974218be2542DAA` |
| MessageTransmitterV2 | `0xE737e5cEBEEBa77EFE34D4aa090756590b1CE275` |
| TokenMinterV2 | `0xb43db544E2c27092c107639Ad201b3dEfAbcF192` |
| MessageV2 | `0xbaC0179bB358A8936169a63408C8481D582390C4` |

**Arc CCTP Domain ID: 26**

### Gateway

| Contract | Address |
|----------|---------|
| GatewayWallet | `0x0077777d7EBA4688BDeF3E311b846F25870A19B9` |
| GatewayMinter | `0x0022222ABE238Cc2C7Bb1f21003F0a260052475B` |

### Payments & Settlement

| Contract | Address |
|----------|---------|
| FxEscrow | `0x867650F5eAe8df91445971f14d89fd84F0C9a9f8` |

### Common Ethereum Predeployed Contracts

| Contract | Address |
|----------|---------|
| CREATE2 Factory | `0x4e59b44847b379578588920cA78FbF26c0B4956C` |
| Multicall3 | `0xcA11bde05977b3631167028862bE2a173976CA11` |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` |

---

## 2.3 EVM Compatibility

### EVM Target

| Property | Value |
|----------|-------|
| EVM Hard Fork Target | **Prague** |
| Consensus Model | Malachite (Tendermint-based BFT) |
| Validator Type | Permissioned |
| Finality | Deterministic, instant (< 1s) |
| Block Timestamps | Wall-clock time, second-level granularity |

**Note:** Multiple blocks may share identical timestamps.

### Native Token & Gas

| Property | Value |
|----------|-------|
| Native Gas Currency | USDC (not ETH) |
| Native Decimal Representation | 18 decimals |
| ERC-20 Interface | Available with 6 decimals |
| Minimum ERC-20 Transfer | > 1 x 10^-6 USDC |
| Fee Mechanism | EWMA-smoothed EIP-1559, bounded base fees |

### Opcode & Feature Differences from Ethereum

| Feature | Status | Notes |
|---------|--------|-------|
| `SELFDESTRUCT` | **Restricted** | Prohibited during deployment to prevent native token burning |
| `PARENT_BEACON_BLOCK_ROOT` | **Modified** | Returns `keccak256(RLP(header))` of parent execution payload; no beacon chain |
| `PREV_RANDAO` | **Disabled** | Always returns `0`; unsuitable as randomness source |
| EIP-4844 Blobs | **Disabled** | Currently unsupported |

### USDC Blocklist Enforcement

| Stage | Behavior |
|-------|----------|
| Pre-mempool | Blocklisted senders rejected before entry; no fees charged |
| Post-mempool | Transactions revert at runtime if address blocklisted; gas consumed |
| Transfer-level | Only the USDC operation reverts; other operations + fees still apply |

### Developer Notes
- No `block.prevrandao` randomness - use external oracles/VRF
- Gas accounting requires USD-denominated display logic
- Standard Solidity, Foundry, and Hardhat fully compatible

---

## 2.4 Gas and Fees

### Gas Parameters (Testnet)

| Parameter | Value |
|-----------|-------|
| Native Gas Token | USDC (18 decimal places) |
| Minimum Base Fee | ~160 Gwei |
| Target Transaction Cost | ~$0.01 |
| Pricing Model | EIP-1559-like with EWMA smoothing |

### Fee Calculation

```
fee = gas_units * base_fee (in USDC with 18 decimals)
```

### Transaction Submission Requirements
- Set `maxFeePerGas >= 160 Gwei` to ensure timely execution
- Transactions with max fees below 160 Gwei risk remaining pending or failing
- Fetch base fees dynamically when submitting transactions

### Dynamic Adjustment
- Exponentially weighted moving-average (EWMA) smoothing
- Stabilizes pricing around 160 Gwei target
- Automatically adjusts based on network load
- Bounded range prevents extreme spikes

### Gas Tracker
Monitor current fees at: https://testnet.arcscan.app/gas-tracker

### Display Guidance
- Display gas costs to users in USDC terms (not Gwei)
- Example: 21,000 gas * 160 Gwei = 0.00000336 USDC (~$0.000003)

> **Note:** These specifications apply to Arc Testnet and may change for mainnet.

---

## 2.5 Sample Applications

### Arc Commerce
- **Repository:** https://github.com/circlefin/arc-commerce
- **Description:** Integrate USDC payments for credit purchases
- **Tech Stack:** Circle Developer Controlled Wallets, Next.js, Supabase
- **Features:** USDC payment integration, credit purchase flow

### Arc Multi-chain Wallet
- **Repository:** https://github.com/circlefin/arc-multichain-wallet
- **Description:** Unified USDC balance and cross-chain transfers
- **Tech Stack:** Circle Gateway, Next.js, Supabase
- **Features:** Cross-chain transfer capability, unified USDC balance management

---

# Part 3: Tools

## 3.1 Account Abstraction

Arc supports **ERC-4337** Account Abstraction, enabling smart contract wallets that initiate and validate transactions without EOAs.

### Architecture
Arc's AA ecosystem is **modular** - developers can mix SDKs, paymasters, and bundlers from multiple providers.

### Supported Token Standards
- ERC-20
- ERC-721
- ERC-1155

### Provider Ecosystem

| Provider | Key Capabilities |
|----------|-----------------|
| **Biconomy** | Modular smart accounts, paymasters, bundlers |
| **Blockradar** | Smart account management APIs, transaction bundling |
| **Circle Wallets** | Cryptographic key management, stablecoin/token support |
| **Crossmint** | Wallet-as-a-service, email/OAuth onboarding |
| **Dynamic** | Passkey wallets, ERC-4337, signer management |
| **Para** | Wallet management, transaction signing infrastructure |
| **Pimlico** | ERC-4337 bundler, paymaster infrastructure, relay services |
| **Privy** | Embedded wallets, key management APIs, email/social auth |
| **Thirdweb** | Full-stack toolkit, managed smart wallet layer |
| **Turnkey** | Programmable key management, policy-based account control |
| **Zerodev** | ERC-4337 deployment, session key support, bundler integration |

---

## 3.2 Block Explorers

### Blockscout (Primary Explorer)

| Property | Value |
|----------|-------|
| Type | Open-source explorer |
| Testnet URL | https://testnet.arcscan.app |
| Provider URL | https://www.blockscout.com |

### Features
- Contract verification
- Token tracking
- Etherscan-compatible APIs for standard queries
- USDC-based fee activity tracking
- Smart contract interaction and verification

---

## 3.3 Compliance Vendors

### Elliptic
- **URL:** https://www.elliptic.co
- **Capabilities:** Blockchain analytics, transaction monitoring APIs, illicit activity identification, risk exposure assessment
- **Functions:** AML and sanctions compliance

### TRM Labs
- **URL:** https://www.trmlabs.com
- **Capabilities:** Risk intelligence, wallet screening, real-time monitoring
- **Functions:** Fraud detection, money laundering detection, suspicious behavior analysis

### Integration Points
- Payment flows
- Wallet infrastructure
- Smart contracts

---

## 3.4 Data Indexers

### Envio
- Event-driven data architecture
- **HyperIndex** for rapid production-grade API development
- Real-time blockchain event streaming with minimal latency

### Goldsky
- Managed subgraph and pipeline service
- Autoscaling with 99.9%+ uptime and up to 6x faster performance
- **Mirror** product streams Arc data to databases at sub-second latency

### The Graph
- Decentralized indexing protocol
- Subgraph capabilities for querying smart contract data
- Graph Explorer for discovering published subgraphs

### Thirdweb
- Open-source blockchain data tooling
- **Insight** tool retrieves Arc data with metadata enrichment and custom transformations

---

## 3.5 Node Providers

### Available Providers

| Provider | Description |
|----------|-------------|
| **Alchemy** | Developer platform with enhanced APIs, monitoring, debugging tools |
| **Blockdaemon** | Institutional-grade node provider, secure and compliant infrastructure |
| **dRPC** | Decentralized RPC aggregator, high-speed load-balanced access |
| **QuickNode** | High-performance blockchain infrastructure, global endpoints and APIs |

### Connection
Connect directly to Arc's public RPC endpoint or through any infrastructure partner using preferred SDK or web3 client.

See [Section 2.1 Connect to Arc](#21-connect-to-arc) for specific endpoint URLs.

---

# Part 4: Tutorials

## 4.1 Deploy on Arc (Foundry)

### Prerequisites
- Foundry toolchain (forge, cast, anvil, chisel)

### Step 1: Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

### Step 2: Initialize Project

```bash
forge init hello-arc && cd hello-arc
```

### Step 3: Environment Setup

Create `.env`:
```bash
ARC_TESTNET_RPC_URL="https://rpc.testnet.arc.network"
PRIVATE_KEY="0x..."
HELLOARCHITECT_ADDRESS="0x..."   # filled after deployment
```

Load: `source .env`

### Step 4: Smart Contract (HelloArchitect.sol)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

contract HelloArchitect {
    string private greeting;

    event GreetingChanged(string newGreeting);

    constructor() {
        greeting = "Hello Architect!";
    }

    function setGreeting(string memory _greeting) public {
        greeting = _greeting;
        emit GreetingChanged(_greeting);
    }

    function getGreeting() public view returns (string memory) {
        return greeting;
    }
}
```

### Step 5: Test

```bash
forge test
```

Test file (HelloArchitect.t.sol) validates:
- Initial greeting correctness
- Greeting update functionality
- Event emission on changes

### Step 6: Build

```bash
forge build
# Generates /out directory with bytecode and ABI
```

### Step 7: Generate Wallet

```bash
cast wallet new
```

### Step 8: Fund Wallet

Visit https://faucet.circle.com, select Arc Testnet, provide wallet address.

### Step 9: Deploy

```bash
forge create src/HelloArchitect.sol:HelloArchitect \
  --rpc-url $ARC_TESTNET_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast
```

### Step 10: Interact

```bash
# Read greeting
cast call $HELLOARCHITECT_ADDRESS "getGreeting()(string)" \
  --rpc-url $ARC_TESTNET_RPC_URL
```

### Step 11: Verify
Use https://testnet.arcscan.app with transaction hash.

---

## 4.2 Deploy Contracts (Circle SDK)

Deploys pre-audited smart contract templates via Circle's Smart Contract Platform.

### Prerequisites
- Node.js v22+ or Python 3.x
- Circle Developer Console account (console.circle.com)
- API Key (Standard Key type)
- Entity Secret (64 lowercase alphanumeric characters)

### Project Setup (Node.js)

```bash
mkdir hello-arc && cd hello-arc
npm init -y
npm pkg set type=module
npm install @circle-fin/developer-controlled-wallets @circle-fin/smart-contract-platform
npm install --save-dev tsx typescript @types/node
npx tsc --init
```

**tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "types": ["node"]
  }
}
```

### Project Setup (Python)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install circle-smart-contract-platform circle-developer-controlled-wallets
```

### Environment Variables

```bash
CIRCLE_API_KEY={YOUR_API_KEY}
CIRCLE_ENTITY_SECRET={YOUR_ENTITY_SECRET}
CIRCLE_WEB3_API_KEY={YOUR_API_KEY}
WALLET_ID={WALLET_ID}
WALLET_ADDRESS={WALLET_ADDRESS}
TRANSACTION_ID={TRANSACTION_ID}
CONTRACT_ID={CONTRACT_ID}
```

### Step 1: Create Wallet (SCA type recommended)

**Node.js:**
```typescript
import { initiateDeveloperControlledWalletsClient } from "@circle-fin/developer-controlled-wallets";

const client = initiateDeveloperControlledWalletsClient({
  apiKey: process.env.CIRCLE_API_KEY,
  entitySecret: process.env.CIRCLE_ENTITY_SECRET,
});

const walletSetResponse = await client.createWalletSet({ name: "Wallet Set 1" });
const walletsResponse = await client.createWallets({
  blockchains: ["ARC-TESTNET"],
  count: 1,
  walletSetId: walletSetResponse.data?.walletSet?.id ?? "",
  accountType: "SCA",
});
```

**Python:**
```python
from circle.web3 import utils, developer_controlled_wallets

client = utils.init_developer_controlled_wallets_client(
    api_key=os.getenv("CIRCLE_API_KEY"),
    entity_secret=os.getenv("CIRCLE_ENTITY_SECRET")
)
wallet_sets_api = developer_controlled_wallets.WalletSetsApi(client)
wallets_api = developer_controlled_wallets.WalletsApi(client)

wallet_set = wallet_sets_api.create_wallet_set(
    developer_controlled_wallets.CreateWalletSetRequest.from_dict({"name": "Wallet Set 1"})
)
wallet = wallets_api.create_wallet(
    developer_controlled_wallets.CreateWalletRequest.from_dict({
        "blockchains": ["ARC-TESTNET"],
        "count": 1,
        "walletSetId": wallet_set.data.wallet_set.actual_instance.id,
        "accountType": "SCA"
    })
)
```

**Response:**
```json
{
  "wallets": [{
    "id": "45692c3e-2ffa-5c5b-a99c-61366939114c",
    "state": "LIVE",
    "address": "0xbcf83d3b112cbf43b19904e376dd8dee01fe2758",
    "blockchain": "ARC-TESTNET",
    "accountType": "SCA",
    "scaCore": "circle_6900_singleowner_v3"
  }]
}
```

### Contract Template IDs

| Template | ID |
|----------|-----|
| ERC-20 | `a1b74add-23e0-4712-88d1-6b3009e85a86` |
| ERC-721 | `76b83278-50e2-4006-8b63-5b1a2a814533` |
| ERC-1155 | `aea21da6-0aa2-4971-9a1a-5098842b1248` |
| Airdrop | `13e322f2-18dc-4f57-8eed-4bddfc50f85e` |

### Deploy ERC-20 (Node.js)

```typescript
import { initiateSmartContractPlatformClient } from "@circle-fin/smart-contract-platform";

const circleContractSdk = initiateSmartContractPlatformClient({
  apiKey: process.env.CIRCLE_API_KEY,
  entitySecret: process.env.CIRCLE_ENTITY_SECRET,
});

const response = await circleContractSdk.deployContractTemplate({
  id: "a1b74add-23e0-4712-88d1-6b3009e85a86",
  blockchain: "ARC-TESTNET",
  name: "MyTokenContract",
  walletId: process.env.WALLET_ID,
  templateParameters: {
    name: "MyToken",
    symbol: "MTK",
    defaultAdmin: process.env.WALLET_ADDRESS,
    primarySaleRecipient: process.env.WALLET_ADDRESS,
  },
  fee: { type: "level", config: { feeLevel: "MEDIUM" } },
});
```

### Deploy ERC-20 (Python)

```python
from circle.web3 import utils, smart_contract_platform

scpClient = utils.init_smart_contract_platform_client(
    api_key=os.getenv("CIRCLE_API_KEY"),
    entity_secret=os.getenv("CIRCLE_ENTITY_SECRET")
)
api_instance = smart_contract_platform.TemplatesApi(scpClient)

request = smart_contract_platform.TemplateContractDeploymentRequest.from_dict({
    "blockchain": "ARC-TESTNET",
    "name": "MyTokenContract",
    "walletId": os.getenv("WALLET_ID"),
    "templateParameters": {
        "name": "MyToken", "symbol": "MTK",
        "defaultAdmin": os.getenv("WALLET_ADDRESS"),
        "primarySaleRecipient": os.getenv("WALLET_ADDRESS"),
    },
    "feeLevel": "MEDIUM"
})
response = api_instance.deploy_contract_template("a1b74add-23e0-4712-88d1-6b3009e85a86", request)
```

### Deploy ERC-20 (cURL)

```bash
curl --request POST \
  --url https://api.circle.com/v1/w3s/templates/a1b74add-23e0-4712-88d1-6b3009e85a86/deploy \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{
    "idempotencyKey": "<string>",
    "entitySecretCiphertext": "<string>",
    "blockchain": "ARC-TESTNET",
    "walletId": "<WALLET_ID>",
    "name": "MyTokenContract",
    "templateParameters": {
      "name": "MyToken", "symbol": "MTK",
      "defaultAdmin": "<ADMIN_ADDRESS>",
      "primarySaleRecipient": "<SALE_ADDRESS>"
    },
    "feeLevel": "MEDIUM"
  }'
```

### Template Parameters

**ERC-20:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| name | String | Yes | Token name |
| symbol | String | Yes | Token symbol |
| defaultAdmin | String | Yes | Admin address |
| primarySaleRecipient | String | Yes | Sale proceeds address |
| platformFeeRecipient | String | No | Platform fee address |
| platformFeePercent | Float | No | Platform fee decimal |
| contractUri | String | No | Metadata URL |
| trustedForwarders | String[] | No | ERC2771 forwarders |

**ERC-721 (additional):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| royaltyRecipient | String | Yes | Secondary sale royalty address |
| royaltyPercent | Float | Yes | Royalty share (e.g., 0.01 = 1%) |

**ERC-1155:** Same parameters as ERC-721.

**Airdrop:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| defaultAdmin | String | Yes | Administrator address |
| contractURI | String | No | Metadata URL |

### Deployment Response

```json
{
  "contractIds": ["019c053d-1ed1-772b-91a8-6970003dad8d"],
  "transactionId": "5b6185b2-f9a1-5645-9db2-ca5d9a330794"
}
```

### Check Transaction Status

```typescript
const transactionResponse = await circleDeveloperSdk.getTransaction({
  id: process.env.TRANSACTION_ID!,
});
```

**Transaction Response (Complete):**
```json
{
  "transaction": {
    "id": "601a0815-f749-41d8-b193-22cadd2a8977",
    "blockchain": "ARC-TESTNET",
    "state": "COMPLETE",
    "contractAddress": "0x281156899e5bd6fecf1c0831ee24894eeeaea2f8",
    "txHash": "0x3bfbab5d...",
    "blockHeight": 23686153,
    "networkFee": "0.044628774800664",
    "operation": "CONTRACT_EXECUTION",
    "feeLevel": "MEDIUM",
    "estimatedFee": {
      "gasLimit": "500797",
      "baseFee": "160",
      "priorityFee": "9.60345525",
      "maxFee": "329.60345525"
    }
  }
}
```

### API Endpoints

| Operation | Endpoint |
|-----------|----------|
| Deploy Template | `POST /v1/w3s/templates/{id}/deploy` |
| Check Transaction | `GET /v1/w3s/transactions/{id}` |
| Get Contract | `GET /v1/w3s/contracts/{id}` |
| Create Wallet | `POST /v1/w3s/wallets` |
| Create Wallet Set | `POST /v1/w3s/wallet-sets` |

### Key Concepts
- **SCA (Smart Contract Accounts):** Recommended for Gas Station compatibility
- **Gas Station:** Automatically sponsors transaction fees for SCA wallets
- **Transaction States:** PENDING -> COMPLETE (10-30 seconds)

---

## 4.3 Interact with Contracts

### SDK Initialization (Common Pattern)

**Node.js:**
```typescript
import { initiateDeveloperControlledWalletsClient } from "@circle-fin/developer-controlled-wallets";

const circleDeveloperSdk = initiateDeveloperControlledWalletsClient({
  apiKey: process.env.CIRCLE_API_KEY,
  entitySecret: process.env.CIRCLE_ENTITY_SECRET,
});
```

**Python:**
```python
from circle.web3 import utils, developer_controlled_wallets

client = utils.init_developer_controlled_wallets_client(
    api_key=os.getenv("CIRCLE_API_KEY"),
    entity_secret=os.getenv("CIRCLE_ENTITY_SECRET")
)
api_instance = developer_controlled_wallets.TransactionsApi(client)
```

### ERC-20 Operations

**Mint:**
```typescript
await circleDeveloperSdk.createContractExecutionTransaction({
  walletId: process.env.WALLET_ID,
  abiFunctionSignature: "mintTo(address,uint256)",
  abiParameters: [process.env.WALLET_ADDRESS, "1000000000000000000"],  // 1 token (18 decimals)
  contractAddress: process.env.CONTRACT_ADDRESS,
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```
> Note: Wallet must have `MINTER_ROLE`

**Transfer:**
```typescript
await circleDeveloperSdk.createContractExecutionTransaction({
  walletId: process.env.WALLET_ID,
  abiFunctionSignature: "transfer(address,uint256)",
  abiParameters: [process.env.RECIPIENT_WALLET_ADDRESS, "1000000000000000000"],
  contractAddress: process.env.CONTRACT_ADDRESS,
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

### ERC-721 Operations

**Mint (with IPFS metadata):**
```typescript
await circleDeveloperSdk.createContractExecutionTransaction({
  walletId: process.env.WALLET_ID,
  abiFunctionSignature: "mintTo(address,string)",
  abiParameters: [
    process.env.WALLET_ADDRESS,
    "ipfs://bafkreibdi6623n3xpf7ymk62ckb4bo75o3qemwkpfvp5i25j66itxvsoei"
  ],
  contractAddress: process.env.CONTRACT_ADDRESS,
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

**Transfer:**
- Signature: `safeTransferFrom(address,address,uint256)`
- Parameters: from address, to address, token ID

### ERC-1155 Operations

**Mint (first token - requires max uint256):**
```typescript
await circleDeveloperSdk.createContractExecutionTransaction({
  walletId: process.env.WALLET_ID,
  abiFunctionSignature: "mintTo(address,uint256,string,uint256)",
  abiParameters: [
    process.env.WALLET_ADDRESS,
    "115792089237316195423570985008687907853269984665640564039457584007913129639935",  // max uint256
    "ipfs://bafkreibdi6623n3xpf7ymk62ckb4bo75o3qemwkpfvp5i25j66itxvsoei",
    "1"
  ],
  contractAddress: process.env.CONTRACT_ADDRESS,
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

> **Critical:** First mint requires maximum uint256 value to create token ID 0. Subsequent mints use "0" for auto-increment.

**Batch Transfer:**
- Signature: `safeBatchTransferFrom(address,address,uint256[],uint256[],bytes)`
- Parameters: from, to, token ID array, amount array, `0x` (empty bytes)

### Airdrop Operations

**Prerequisites:** Deployed token contract, token balance, token approval (`approve` or `setApprovalForAll`)

**ERC-20 Airdrop:**
```typescript
abiFunctionSignature: "airdropERC20(address,(address,uint256)[])",
abiParameters: [
  "<TOKEN_CONTRACT_ADDRESS>",
  [
    ["<RECIPIENT_1>", "1000000000000000000"],
    ["<RECIPIENT_2>", "2000000000000000000"]
  ]
]
```

**ERC-721 Airdrop:**
```typescript
abiFunctionSignature: "airdropERC721(address,(address,uint256)[])",
abiParameters: [
  "<TOKEN_CONTRACT_ADDRESS>",
  [["<RECIPIENT_1>", "1"], ["<RECIPIENT_2>", "2"]]
]
```

**ERC-1155 Airdrop:**
```typescript
abiFunctionSignature: "airdropERC1155(address,(address,uint256,uint256)[])",
abiParameters: [
  "<TOKEN_CONTRACT_ADDRESS>",
  [["<RECIPIENT_1>", "0", "10"], ["<RECIPIENT_2>", "1", "5"]]
]
```

### API Endpoint

```
POST https://api.circle.com/v1/w3s/developer/transactions/contractExecution
Headers:
  authorization: Bearer <API_KEY>
  content-type: application/json
Body:
  idempotencyKey, entitySecretCiphertext, walletId,
  abiFunctionSignature, abiParameters, contractAddress, feeLevel
```

---

## 4.4 Monitor Contract Events

### Two Approaches

| Approach | Model | Use Case |
|----------|-------|----------|
| Webhooks | Push (real-time) | Production, low latency |
| Polling | Pull (periodic) | Testing, historical queries |

### Step 1: Set Up Webhook Endpoint

**Option A: Webhook.site** - Use directly, no setup needed.

**Option B: ngrok + Express.js**

```typescript
import express from "express";
const app = express();
app.use(express.json());

app.post("/webhook", (req, res) => {
  console.log("Received webhook:", JSON.stringify(req.body, null, 2));
  res.status(200).json({ received: true });
});

app.listen(3000);
```

```bash
ngrok http 3000
# Copy HTTPS forwarding URL
```

**Option B: ngrok + Flask**

```python
from flask import Flask, request, jsonify
app = Flask(__name__)

@app.route("/webhook", methods=["POST"])
def webhook():
    data = request.get_json()
    print("Received webhook:", data)
    return jsonify({"received": True}), 200

app.run(port=3000)
```

### Step 2: Register Webhook in Developer Console

Navigate to Webhooks section -> "Add a webhook" -> Enter URL.
**Must complete before creating monitors.**

### Step 3: Import Contract (Optional)

```typescript
const response = await contractClient.importContract({
  blockchain: "ARC-TESTNET",
  address: process.env.CONTRACT_ADDRESS,
  name: "MyContract",
});
```

### Step 4: Create Event Monitor

```typescript
const response = await contractClient.createEventMonitor({
  blockchain: "ARC-TESTNET",
  contractAddress: process.env.CONTRACT_ADDRESS,
  eventSignature: "Transfer(address,address,uint256)",
});
```

**Response:**
```json
{
  "eventMonitor": {
    "id": "019bf984-b4da-7026-a3d2-674ce371a933",
    "contractAddress": "0x281156899e5bd6fecf1c0831ee24894eeeaea2f8",
    "blockchain": "ARC-TESTNET",
    "eventSignature": "Transfer(address,address,uint256)",
    "eventSignatureHash": "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
    "isEnabled": true
  }
}
```

### Webhook Notification Payload

```json
{
  "subscriptionId": "f0332621-...",
  "notificationId": "5c5eea9f-...",
  "notificationType": "contracts.eventLog",
  "notification": {
    "contractAddress": "0x4abcffb90897fe7ce86ed689d1178076544a021b",
    "blockchain": "ARC-TESTNET",
    "txHash": "0xe15d6dbb...",
    "userOpHash": "0x78c3e818...",
    "blockHash": "0x0ad6bf57...",
    "blockHeight": 22807198,
    "eventSignature": "Transfer(address,address,uint256)",
    "eventSignatureHash": "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
    "topics": [
      "0xddf252ad...",     // event signature hash
      "0x000000...0000",   // from address (indexed)
      "0x000000...e2758"   // to address (indexed)
    ],
    "data": "0x0000...640000",  // amount (non-indexed)
    "firstConfirmDate": "2026-01-21T06:53:12Z"
  },
  "timestamp": "2026-01-21T06:53:13.194467201Z",
  "version": 2
}
```

### Step 5: Poll Event Logs (Alternative)

```typescript
const response = await contractClient.listEventLogs({
  contractAddress: process.env.CONTRACT_ADDRESS,
  blockchain: "ARC-TESTNET",
  pageSize: 10,
});
```

---

## 4.5 Bridge USDC to Arc

Uses Circle's **Bridge Kit** and **CCTP** (Cross-Chain Transfer Protocol) for native burning and minting.

### Supported Source Chains (Testnet)

| Chain | Network |
|-------|---------|
| Arbitrum Sepolia | Testnet |
| Avalanche Fuji | Testnet |
| Base Sepolia | Testnet |
| Ethereum Sepolia | Testnet |
| Optimism Sepolia | Testnet |
| Polygon Amoy | Testnet |
| Solana Devnet | Testnet |
| Unichain Sepolia | Testnet |

**Destination:** Arc_Testnet

### Required Dependencies

```bash
npm install @circle-fin/bridge-kit @circle-fin/adapter-circle-wallets @circle-fin/developer-controlled-wallets
```

### Bridge Transaction 4-Phase Process

```
1. APPROVE     --> Authorize token spending on source chain
2. BURN        --> Destroy USDC on source chain
3. ATTESTATION --> Retrieve CCTP attestation from Circle
4. MINT        --> Create USDC on Arc (destination)
```

### Bridge Function Parameters

```typescript
{
  from: {
    chain: "<SOURCE_CHAIN>",
    adapter: walletAdapter,
    address: "<SOURCE_ADDRESS>"
  },
  to: {
    chain: "Arc_Testnet",
    adapter: walletAdapter,
    address: "<DESTINATION_ADDRESS>"
  },
  amount: "1.00"
}
```

### Setup Requirements
- Node.js v22+
- Circle Developer Console account
- API key + Entity Secret
- Dev-controlled wallets on both chains
- Testnet USDC + native tokens for gas on source chain

### Verification
Use `explorerUrl` fields from bridge response steps to verify on respective block explorers.

---

## 4.6 Transfer USDC or EURC

### Token Addresses (Arc Testnet)

| Token | Address |
|-------|---------|
| USDC | `0x3600000000000000000000000000000000000000` |
| EURC | `0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a` |

### Setup

```bash
mkdir transfer-funds && cd transfer-funds
npm init -y && npm pkg set type=module
npm install @circle-fin/developer-controlled-wallets
```

### USDC Transfer (Node.js)

```typescript
const transferResponse = await client.createTransaction({
  amount: ["0.1"],
  destinationAddress: "<RECIPIENT_ADDRESS>",
  tokenAddress: "0x3600000000000000000000000000000000000000",
  blockchain: "ARC-TESTNET",
  walletAddress: "<SENDER_ADDRESS>",
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

### EURC Transfer (Node.js)

```typescript
const transferResponse = await client.createTransaction({
  amount: ["0.1"],
  destinationAddress: "<RECIPIENT_ADDRESS>",
  tokenAddress: "0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a",
  blockchain: "ARC-TESTNET",
  walletAddress: "<SENDER_ADDRESS>",
  fee: { type: "level", config: { feeLevel: "MEDIUM" } }
});
```

### Transfer (Python)

```python
request = developer_controlled_wallets.CreateTransferTransactionForDeveloperRequest.from_dict({
    "amounts": ['0.1'],
    "destinationAddress": "<RECIPIENT_ADDRESS>",
    "tokenAddress": "0x3600000000000000000000000000000000000000",
    "blockchain": "ARC-TESTNET",
    "walletAddress": "<SENDER_ADDRESS>",
    "feeLevel": 'MEDIUM'
})
response = api_instance.create_developer_transaction_transfer(request)
```

### Transfer (cURL)

```bash
curl --request POST \
  --url https://api.circle.com/v1/w3s/developer/transactions/transfer \
  --header 'Authorization: Bearer <YOUR_API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{
    "idempotencyKey": "<string>",
    "entitySecretCiphertext": "<string>",
    "amounts": ["0.1"],
    "destinationAddress": "<RECIPIENT>",
    "tokenAddress": "<TOKEN_ADDRESS>",
    "blockchain": "ARC-TESTNET",
    "walletAddress": "<SENDER>",
    "feeLevel": "MEDIUM"
  }'
```

### Check Transaction Status

```typescript
const response = await client.getTransaction({ id: "<TRANSACTION_ID>" });
```

```bash
curl --request GET \
  --url https://api.circle.com/v1/w3s/transactions/{TRANSACTION_ID} \
  --header 'Authorization: Bearer <YOUR_API_KEY>'
```

### Verification
- Check `state` field for "COMPLETE"
- View tx hash on https://testnet.arcscan.app

---

## 4.7 Access USDC Crosschain (Gateway)

Chain-abstraction for USDC using Circle Gateway - unified balance management across multiple blockchains.

### Supported Chains & Domain IDs

| Chain | Domain ID | USDC Address | Network ID |
|-------|-----------|--------------|------------|
| Ethereum Sepolia | 0 | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` | ETH-SEPOLIA |
| Avalanche Fuji | 1 | `0x5425890298aed601595a70AB815c96711a31Bc65` | AVAX-FUJI |
| Base Sepolia | 6 | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` | BASE-SEPOLIA |
| Arc Testnet | 26 | `0x3600000000000000000000000000000000000000` | ARC-TESTNET |

### Gateway Contract Addresses

| Contract | Address |
|----------|---------|
| GatewayWallet | `0x0077777d7EBA4688BDeF3E311b846F25870A19B9` |
| GatewayMinter | `0x0022222ABE238Cc2C7Bb1f21003F0a260052475B` |

### API Endpoints

| Operation | Endpoint |
|-----------|----------|
| Balance Query | `POST https://gateway-api-testnet.circle.com/v1/balances` |
| Transfer/Attestation | `POST https://gateway-api-testnet.circle.com/v1/transfer` |

### Project Setup

```bash
npm init -y && npm pkg set type=module
npm install @circle-fin/developer-controlled-wallets
npm install --save-dev tsx typescript @types/node
```

### Gateway Flow (5 Steps)

#### Step 1: Create Wallets
Create wallets across supported chains using same `refId` for identical addresses.

#### Step 2: Deposit into Gateway

**Phase A - Approve USDC:**
```typescript
abiFunctionSignature: "approve(address,uint256)"
// Approve Gateway Wallet address for USDC spend
```

**Phase B - Deposit:**
```typescript
abiFunctionSignature: "deposit(address,uint256)"
// Call deposit() on Gateway Wallet contract
// IMPORTANT: Use deposit(), NOT standard transfer()
```

**Critical:** Must call `deposit` method on Gateway Wallet contract - NOT `transfer` on USDC contract.

#### Step 3: Check Balance

Query `POST https://gateway-api-testnet.circle.com/v1/balances` with:
- Token type
- Array of source domains with depositor addresses

Returns per-domain balances; total = unified balance across chains.

#### Step 4: Transfer (Burn Intent + Attestation)

**4a. Construct EIP-712 Typed Data (BurnIntent):**

```typescript
// TransferSpec structure
{
  version: uint32,
  sourceDomain: uint32,       // e.g., 0 for Ethereum
  destinationDomain: uint32,  // e.g., 26 for Arc
  sourceContract: bytes32,
  destinationContract: bytes32,
  sourceToken: bytes32,
  destinationToken: bytes32,
  sourceDepositor: bytes32,
  destinationRecipient: bytes32,
  sourceSigner: bytes32,
  destinationCaller: bytes32,
  value: uint256,
  salt: bytes32,
  hookData: bytes
}

// BurnIntent wrapper
{
  maxBlockHeight: uint256,
  maxFee: uint256,           // typical: 2,010,000 base units
  spec: TransferSpec
}
```

**4b. Sign Intent:**
```typescript
wallet.signTypedData(typedData)
```

**4c. Submit to Gateway API:**
```
POST https://gateway-api-testnet.circle.com/v1/transfer
```
Returns attestation + operator signature.

**4d. Mint on Destination:**
```typescript
abiFunctionSignature: "gatewayMint(bytes,bytes)"
// Execute on GatewayMinter contract with attestation + operator signature
```

### Utility Functions

**Balance Parsing (decimal to base units):**
Convert "10.5" to BigInt with 6-decimal representation (10,500,000)

**Address Formatting:**
Pad Ethereum addresses to 32 bytes (bytes32) for EIP-712 compliance

**Transaction Polling:**
Poll interval: 3 seconds. Terminal states: COMPLETE, CONFIRMED, FAILED, DENIED, CANCELLED.

### Operational Notes
- Block confirmation times vary: up to 19-20 minutes for finality on some chains
- Gas fees differ per destination chain
- Per-burn-intent fee structure
- Recipient address can differ from depositor

---

# Quick Reference Card

## Network Configuration

```
Chain ID:        5042002
RPC:             https://rpc.testnet.arc.network
WSS:             wss://rpc.testnet.arc.network
Explorer:        https://testnet.arcscan.app
Faucet:          https://faucet.circle.com
Gas Token:       USDC (18 decimals native, 6 decimals ERC-20)
Min Base Fee:    160 Gwei
CCTP Domain:     26
EVM Target:      Prague
```

## 4.8 Register Your First AI Agent (ERC-8004)

> Source: https://docs.arc.network/arc/tutorials/register-your-first-ai-agent

Register AI agents with onchain identity using the **ERC-8004** standard. Covers identity registration, reputation tracking, and credential validation on Arc Testnet using Circle's Developer-Controlled Wallets API.

### ERC-8004 Contract Addresses

| Contract | Address | Purpose |
|----------|---------|---------|
| IdentityRegistry | `0x8004A818BFB912233c491871b3d84c89A494BD9e` | Agent identity (ERC-721 NFT) |
| ReputationRegistry | `0x8004B663056A597Dffe9eCcC1965A193B7388713` | Reputation scoring |
| ValidationRegistry | `0x8004Cb1BF31DAf7788923b405b754f57acEB4272` | Credential validation |

### Prerequisites
- Circle Developer Console account + API Key (Standard Key)
- Registered Entity Secret

### Setup
```bash
npm install @circle-fin/developer-controlled-wallets viem
```

### Step 1: Create Two Developer-Controlled Wallets

Two wallets required: **owner** (registers agent) and **validator** (records reputation). Per ERC-8004: "agent owners cannot record reputation for their own agents to prevent self-dealing."

```javascript
const walletSet = await circleClient.createWalletSet({ name: "ERC8004 Agent Wallets" });
const wallets = await circleClient.createWallets({
  blockchains: ["ARC-TESTNET"], count: 2,
  walletSetId: walletSet.data?.walletSet?.id, accountType: "SCA",
});
const ownerWallet = wallets.data?.wallets?.[0];
const validatorWallet = wallets.data?.wallets?.[1];
```

### Step 2: Prepare Agent Metadata

Upload JSON metadata to IPFS (name, description, capabilities, version). Test URI: `ipfs://bafkreibdi6623n3xpf7ymk62ckb4bo75o3qemwkpfvp5i25j66itxvsoei`

### Step 3: Register Agent Identity

Call `register(string metadataURI)` on IdentityRegistry → mints ERC-721 identity NFT.

```javascript
await circleClient.createContractExecutionTransaction({
  walletAddress: ownerWallet.address, blockchain: "ARC-TESTNET",
  contractAddress: "0x8004A818BFB912233c491871b3d84c89A494BD9e",
  abiFunctionSignature: "register(string)",
  abiParameters: [METADATA_URI],
  fee: { type: "level", config: { feeLevel: "MEDIUM" } },
});
```

### Step 4: Retrieve Agent ID

Query Transfer events from IdentityRegistry to find the minted tokenId.

### Step 5: Record Reputation

Validator wallet calls `giveFeedback(uint256 agentId, int128 score, uint8 feedbackType, string tag, ...)` on ReputationRegistry.

```javascript
// score: 95, feedbackType: 0, tag: "successful_trade"
await circleClient.createContractExecutionTransaction({
  walletAddress: validatorWallet.address,
  contractAddress: "0x8004B663056A597Dffe9eCcC1965A193B7388713",
  abiFunctionSignature: "giveFeedback(uint256,int128,uint8,string,string,string,string,bytes32)",
  abiParameters: [agentId, "95", "0", tag, "", "", "", feedbackHash],
  ...
});
```

### Step 6: Request Validation

Owner calls `validationRequest(address validator, uint256 agentId, string requestURI, bytes32 requestHash)` on ValidationRegistry.

### Step 7: Validator Response

Validator calls `validationResponse(bytes32 requestHash, uint8 response, string responseURI, bytes32 responseHash, string tag)`. Response: **100 = passed, 0 = failed**.

### Step 8: Verify Validation Status

Query `getValidationStatus(bytes32 requestHash)` → returns (validatorAddress, agentId, response, responseHash, tag, lastUpdate).

### Key Rules
- **Owner ≠ Validator**: Cannot self-report reputation (anti-self-dealing)
- **Score range**: int128 for reputation, uint8 for validation (100=pass, 0=fail)
- **Gas**: ~0.006 USDC per transaction with Circle Gas Station
- **Cast wallet alternative**: Can use EOA wallets with `cast send` instead of Circle API; need a second EOA for validator (transfer 1 USDC for gas)

### Function Signatures

```
IdentityRegistry:
  register(string metadataURI) → mints ERC-721

ReputationRegistry:
  giveFeedback(uint256 agentId, int128 score, uint8 feedbackType,
               string tag, string comment, string uri, string metadata, bytes32 hash)

ValidationRegistry:
  validationRequest(address validator, uint256 agentId, string requestURI, bytes32 requestHash)
  validationResponse(bytes32 requestHash, uint8 response, string responseURI, bytes32 responseHash, string tag)
  getValidationStatus(bytes32 requestHash) → (address, uint256, uint8, bytes32, string, uint256)
```

---

# Quick Reference

## Key Addresses

```
USDC:              0x3600000000000000000000000000000000000000
EURC:              0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a
TokenMessengerV2:  0x8FE6B999Dc680CcFDD5Bf7EB0974218be2542DAA
MessageTransmitter:0xE737e5cEBEEBa77EFE34D4aa090756590b1CE275
TokenMinterV2:     0xb43db544E2c27092c107639Ad201b3dEfAbcF192
GatewayWallet:     0x0077777d7EBA4688BDeF3E311b846F25870A19B9
GatewayMinter:     0x0022222ABE238Cc2C7Bb1f21003F0a260052475B
IdentityRegistry:  0x8004A818BFB912233c491871b3d84c89A494BD9e  (ERC-8004)
ReputationRegistry:0x8004B663056A597Dffe9eCcC1965A193B7388713  (ERC-8004)
ValidationRegistry:0x8004Cb1BF31DAf7788923b405b754f57acEB4272  (ERC-8004)
FxEscrow:          0x867650F5eAe8df91445971f14d89fd84F0C9a9f8
CREATE2 Factory:   0x4e59b44847b379578588920cA78FbF26c0B4956C
Multicall3:        0xcA11bde05977b3631167028862bE2a173976CA11
Permit2:           0x000000000022D473030F116dDEE9F6B43aC78BA3
```

## Circle API Base URLs

```
Web3 Services:     https://api.circle.com/v1/w3s/
Gateway Testnet:   https://gateway-api-testnet.circle.com/v1/
```

## Template IDs

```
ERC-20:   a1b74add-23e0-4712-88d1-6b3009e85a86
ERC-721:  76b83278-50e2-4006-8b63-5b1a2a814533
ERC-1155: aea21da6-0aa2-4971-9a1a-5098842b1248
Airdrop:  13e322f2-18dc-4f57-8eed-4bddfc50f85e
```
