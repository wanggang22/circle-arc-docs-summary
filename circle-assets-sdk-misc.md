# Circle Developer Documentation - Group 8: Miscellaneous (Comprehensive Summary)

> **Source**: 57 pages from developers.circle.com
> **Generated**: 2026-03-04

---

## Table of Contents

1. [Platform Overview & Products](#1-platform-overview--products)
2. [Assets: USDC](#2-assets-usdc)
3. [Assets: EURC](#3-assets-eurc)
4. [Assets: USYC (Tokenized Yield)](#4-assets-usyc-tokenized-yield)
5. [xReserve (USDC-Backed Stablecoins)](#5-xreserve-usdc-backed-stablecoins)
6. [Stablecoin Transfers on Various Chains](#6-stablecoin-transfers-on-various-chains)
7. [Smart Contracts Platform](#7-smart-contracts-platform)
8. [Contract Templates](#8-contract-templates)
9. [SDKs](#9-sdks)
10. [AI / MCP Integration](#10-ai--mcp-integration)
11. [Cross-Chain Transfers](#11-cross-chain-transfers)
12. [Liquidity Services](#12-liquidity-services)
13. [Payments Network](#13-payments-network)
14. [Sample Projects](#14-sample-projects)
15. [Release Notes](#15-release-notes)

---

## 1. Platform Overview & Products

### Product Categories

Circle's developer platform organizes into these major areas:

**Assets (Permissionless)**:
- **USDC** - Digital dollar stablecoin for payments and DeFi
- **EURC** - Euro-backed stablecoin (MiCA-compliant)
- **xReserve** - Platform for issuing USDC-backed stablecoins (1:1 reserve)
- **USYC** - Tokenized yield-bearing US Treasury product

**Onchain Application Development**:
- Developer-Controlled Wallets (API-driven)
- User-Controlled Wallets (passkeys, social login)
- Modular Smart Accounts (permissions, recovery, automation modules)
- Gas Station (sponsored gas / gasless UX)
- Paymaster (pay gas in USDC, permissionless ERC-4337)
- Smart Contracts (deploy, manage, monitor)
- Compliance Engine (transaction screening, rules, alerts)

**Cross-Chain Infrastructure**:
- **Bridge Kit** - SDK for USDC transfers across EVM and non-EVM chains
- **CCTP** - Burn-and-mint native USDC between supported chains (permissionless)
- **Gateway** - Unified USDC balance across chains (Circle-managed settlement)
- **Nanopayments** - Gas-free sub-cent USDC payments (min $0.000001, batched settlement)

**Financial Services**:
- Circle Payments Network (CPN) - Cross-border institutional payments
- Circle Mint - Direct USDC/EURC minting and redemption
- StableFX - Institutional FX trading (USDC/EURC)

### Authentication Model
- **Permissioned products**: Require Circle credentials (Wallets, Gas Station, Contracts, etc.)
- **Permissionless products**: Directly integrable (USDC, EURC, CCTP, Gateway, Paymaster, xReserve)

### Key API Endpoints
- Gateway Testnet: `https://gateway-api-testnet.circle.com/v1/info`
- IRIS Sandbox: `https://iris-api-sandbox.circle.com/v2/burn/USDC/fees/`
- Documentation index: `https://developers.circle.com/llms.txt`

---

## 2. Assets: USDC

### What is USDC
- Stablecoin issued by Circle representing US dollars on blockchain networks
- 1:1 redemption ratio with USD
- Backed 100% by highly liquid cash and cash-equivalent assets
- Monthly attestation reports published on Circle's transparency page
- Open, programmable, 24/7/365 availability
- Instant settlement with low transaction costs

### USDC Contract Addresses - Mainnet

| Blockchain | Address |
|---|---|
| Algorand | `31566704` |
| Aptos | `0xbae207659db88bea0cbead6da0ed00aac12edcdda169e591cd41c94180b46f3b` |
| Arbitrum | `0xaf88d065e77c8cC2239327C5EDb3A432268e5831` |
| Avalanche C-Chain | `0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E` |
| Base | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| Codex | `0xd996633a415985DBd7D6D12f4A4343E31f5037cf` |
| Celo | `0xcebA9300f2b948710d2653dD7B07f33A8B32118C` |
| Ethereum | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| Hedera | `0.0.456858` |
| HyperEVM | `0xb88339CB7199b77E23DB6E890353E22632Ba630f` |
| Ink | `0x2D270e6886d130D724215A266106e6832161EAEd` |
| Linea | `0x176211869cA2b568f2A7D4EE941E073a821EE1ff` |
| Monad | `0x754704Bc059F8C67012fEd69BC8A327a5aafb603` |
| NEAR | `17208628f84f5d6ad33f0da3bbbeb27ffcb398eac501a31bd6ad2011e36133a1` |
| Noble | `uusdc` |
| OP Mainnet | `0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85` |
| Plume | `0x222365EF19F7947e5484218551B56bb3965Aa7aF` |
| Polkadot Asset Hub | `1337` |
| Polygon PoS | `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359` |
| Sei | `0xe15fC38F6D8c56aF07bbCBe3BAf5708A2Bf42392` |
| Solana | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| Sonic | `0x29219dd400f2Bf60E5a23d13Be72B486D4038894` |
| Starknet | `0x033068F6539f8e6e6b131e6B2B814e6c34A5224bC66947c47DaB9dFeE93b35fb` |
| Stellar | `USDC-GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN` |
| Sui | `0xdba34672e30cb065b1f93e3ab55318768fd6fef66c15942c9f7cb846e2f900e7::usdc::USDC` |
| Unichain | `0x078D782b760474a361dDA0AF3839290b0EF57AD6` |
| World Chain | `0x79A02482A880bCe3F13E09da970dC34dB4cD24D1` |
| XDC | `0xfA2958CB79b0491CC627c1557F441eF849Ca8eb1` |
| XRPL | `5553444300000000000000000000000000000000.rGm7WCVp9gb4jZHWTEtGUr4dd74z2XuWhE` |
| ZKsync Era | `0x1d17CBcF0D6D143135aE902365D2E5e2A16538D4` |

### USDC Contract Addresses - Testnet

| Blockchain | Address |
|---|---|
| Algorand Testnet | `10458941` |
| Aptos Testnet | `0x69091fbab5f7d635ee7ac5098cf0c1efbe31d68fec0f2cd565e8d168daf52832` |
| Arbitrum Sepolia | `0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d` |
| Arc Testnet | `0x3600000000000000000000000000000000000000` |
| Avalanche Fuji | `0x5425890298aed601595a70AB815c96711a31Bc65` |
| Base Sepolia | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| Celo Sepolia | `0x01C5C0122039549AD1493B8220cABEdD739BC44E` |
| Codex Testnet | `0x6d7f141b6819C2c9CC2f818e6ad549E7Ca090F8f` |
| EDGE Testnet | `0x2d9F7CAD728051AA35Ecdc472a14cf8cDF5CFD6B` |
| Ethereum Sepolia | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` |
| Hedera Testnet | `0.0.429274` |
| HyperEVM Testnet | `0x2B3370eE501B4a559b57D449569354196457D8Ab` |
| Ink Testnet | `0xFabab97dCE620294D2B0b0e46C68964e326300Ac` |
| Linea Sepolia | `0xFEce4462D57bD51A6A552365A011b95f0E16d9B7` |
| Monad Testnet | `0x534b2f3A21130d7a60830c2Df862319e593943A3` |
| Morph Hoodi Testnet | `0x7433b41C6c5e1d58D4Da99483609520255ab661B` |
| NEAR Testnet | `3e2210e1184b45b64c8a434c0a7e7b23cc04ea7eb7a6c3c32520d03d4afcb8af` |
| Noble Testnet | `uusdc` |
| OP Sepolia | `0x5fd84259d66Cd46123540766Be93DFE6D43130D7` |
| Plume Testnet | `0xcB5f30e335672893c7eb944B374c196392C19D18` |
| Polkadot Westmint | `Asset ID 31337` |
| Polygon PoS Amoy | `0x41E94Eb019C0762f9Bfcf9Fb1E58725BfB0e7582` |
| Sei Testnet | `0x4fCF1784B31630811181f670Aea7A7bEF803eaED` |
| Solana Devnet | `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU` |
| Sonic Testnet | `0x0BA304580ee7c9a980CF72e55f5Ed2E9fd30Bc51` |
| Sonic Blaze Testnet | `0xA4879Fed32Ecbef99399e5cbC247E533421C4eC6` |
| Starknet Sepolia | `0x0512feAc6339Ff7889822cb5aA2a86C848e9D392bB0E3E237C008674feeD8343` |
| Stellar Testnet | `USDC-GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5` |
| Sui Testnet | `0xa1ec7fc00a6f40db9693ad1415d0c193ad3906494428cf252621037bd7117e29::usdc::USDC` |
| Unichain Sepolia | `0x31d0220469e10c4E71834a79b1f276d740d3768F` |
| World Chain Sepolia | `0x66145f38cBAC35Ca6F1Dfb4914dF98F1614aeA88` |
| XDC Apothem | `0xb5AB69F7bBada22B28e79C8FFAECe55eF1c771D4` |
| XRPL Testnet | `5553444300000000000000000000000000000000.rHuGNhqTG32mfmAvWA8hUyWRLV3tCSwKQt` |
| ZKsync Era Testnet | `0xAe045DE5638162fa134807Cb558E15A3F5A7F853` |

### Developer Integration Paths
1. **Direct smart contract integration** - Customizable fund flows
2. **Circle Developer Services** - SDK/API suite for blockchain newcomers
3. **CCTP** - Permissionless cross-chain USDC movement

---

## 3. Assets: EURC

### What is EURC
- Circle's digital euro stablecoin
- 100% backed by euro-denominated reserves
- 1:1 redeemable for EUR
- Compliant with EU's Markets in Crypto-Assets (MiCA) regulation
- Monthly attestation reports on reserves

### EURC Contract Addresses - Mainnet

| Blockchain | Contract Address |
|---|---|
| Avalanche C-Chain | `0xC891EB4cbdEFf6e073e859e987815Ed1505c2ACD` |
| Base | `0x60a3E35Cc302bFA44Cb288Bc5a4F316Fdb1adb42` |
| Ethereum | `0x1aBaEA1f7C830bD89Acc67eC4af516284b1bC33c` |
| Solana | `HzwqbKZw8HxMN6bF2yFZNrht3c2iXXzpKcFu7uBEDKtr` |
| Stellar | `EURC-GDHU6WRG4IEQXM5NZ4BMPKOXHW76MZM4Y2IEMFDVXBSDP6SJY4ITNPP2` |
| World Chain | `0x1C60ba0A0eD1019e8Eb035E6daF4155A5cE2380B` |

### EURC Contract Addresses - Testnet

| Network | Contract Address |
|---|---|
| Arc Testnet | `0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a` |
| Avalanche Fuji | `0x5E44db7996c682E92a960b65AC713a54AD815c6B` |
| Base Sepolia | `0x808456652fdb597867f38412077A9182bf77359F` |
| Ethereum Sepolia | `0x08210F9170F89Ab7658F0B5E3fF39b0E03C594D4` |
| Solana Devnet | `HzwqbKZw8HxMN6bF2yFZNrht3c2iXXzpKcFu7uBEDKtr` |
| Stellar Testnet | `EURC-GB3Q6QDZYTHWT7E5PVS3W7FUT5GVAFC5KSZFFLPU25GO7VTC3NM2ZTVO` |
| World Chain Sepolia | `0xe479EcA5740Ac65d6E1823bea2f1C08Bc14e954F` |

---

## 4. Assets: USYC (Tokenized Yield)

### Overview
- **USYC** = onchain representation of Hashnote International Short Duration Yield Fund Ltd.
- Invests primarily in reverse repurchase agreements backed by U.S. government securities
- Yields through the overnight federal funds rate via reverse repo agreements
- **Issuer**: Circle International Bermuda Limited
- **Regulator**: Bermuda Monetary Authority (BMA)
- **Fund Jurisdiction**: Cayman Islands (regulated by CIMA)
- Standard ERC-20 token, DeFi-compatible
- Subscriptions and redemptions settle atomically in real time (T+0), 24/7/365
- Token pricing and fund holdings published via onchain Oracle
- **Eligibility**: Restricted to non-U.S. Persons (Regulation S, Securities Act of 1933)

### USYC Smart Contract Addresses - Mainnet

**Ethereum:**
| Contract | Address |
|---|---|
| USYC Token | `0x136471a34f6ef19fE571EFFC1CA711fdb8E49f2b` |
| Teller (USDC) | `0xeE35F963BFC71b51eC95147f26c030D674ea30e6` |
| Cross Chain Teller (USDC) | `0x5575a88BEC5b47ce8D270F6A4F2418865f16AfD5` |
| Entitlements | `0x902D906b8d988092213bE799B18Bd2cbd64F808C` |
| Oracle | `0x4c48bcb2160F8e0aDbf9D4F3B034f1e36d1f8b3e` |

**BSC:**
| Contract | Address |
|---|---|
| USYC Token | `0x8D0fA28f221eB5735BC71d3a0Da67EE5bC821311` |
| Cross Chain Teller (USDC) | `0xf38979E05650be7926EA07BB59C48Fb9b1DB3D08` |
| Entitlements | `0x6B7d54003f73bE979cf92BF369432aC534853692` |

**Solana:**
| Contract | Address |
|---|---|
| USYC Token | `7LWanZteUKtvFjv4MHYgKXXdAuCQYFPJysL9pxxdRQGn` |
| Teller & Permissions | `EMfvuBR8rre2f7sNqdDyPzQW7kgBUsVtxg7nX93YwAdm` |

### USYC Smart Contract Addresses - Testnet

Available on Ethereum Sepolia, BSC Testnet, Arc Testnet, and Solana Devnet.

**Ethereum Sepolia:**
| Contract | Address |
|---|---|
| Teller Contract | `0x96424C885951ceb4B79fecb934eD857999e6f82B` |
| USDC Token | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` |
| USYC Token | `0x38D3A3f8717F4DB1CcB4Ad7D8C755919440848A3` |

### USYC Teller Smart Contract Functions

**Deposit (Subscribe):**
```solidity
deposit(uint256 _assets, address _receiver) returns (uint256)
```
- `_assets`: USDC amount in smallest unit (6 decimals)
- `_receiver`: Address to receive USYC
- Returns: USYC amount received

**Redeem (Withdraw):**
```solidity
redeem(uint256 _shares, address _receiver, address _account) returns (uint256)
```
- `_shares`: USYC amount to redeem
- `_receiver`: Address to receive USDC
- `_account`: USYC holder address
- Returns: USDC payout amount

Both USDC and USYC use 6 decimal places (100 USDC = `100 * 1e6`).

### USYC Integration Steps (EVM)
1. Approve Teller contract to spend USDC
2. Call `deposit()` with USDC amount and receiver address
3. For redemptions, call `redeem()` with USYC amount, recipient, and account addresses

### USYC on Solana

**Devnet Program Address**: `8FZQeFy39fNK47ebUd7aUWoK1rLgJ7fsteWev3FPP1Aw`

**Dependencies**: `@solana/kit`, `@solana-program/token`, `@solana-program/token-2022`

**Process:**
1. Derive PDAs (teller, event authority) from seeds
2. Fetch teller account data (asset mint, share mint, treasury, fee recipient)
3. Generate deposit/redeem instructions via Yieldcoin Manager IDL
4. Build, sign, and send transaction

### USYC Portal
- **Mainnet**: https://usyc.hashnote.com
- **Testnet**: https://usyc.dev.hashnote.com

### USYC Web2 API Endpoints

**Mainnet:**
- Current price: `https://usyc.hashnote.com/api/price`
- Historical data: `https://usyc.hashnote.com/api/price-reports`
- Entitlement check: `https://api.hashnote.com/v1/entitlements/token_access?address={address}&symbol=USYC`

**Testnet:**
- Current price: `https://usyc.dev.hashnote.com/api/price`
- Historical data: `https://usyc.dev.hashnote.com/api/price-reports`
- Entitlement check: `https://api.dev.hashnote.com/v1/entitlements/token_access?address={address}&symbol=USYC`

---

## 5. xReserve (USDC-Backed Stablecoins)

### Overview
xReserve is an interoperability platform enabling blockchain networks to launch **USDC-backed stablecoins (USDCx)** independently. It uses programmatic attestations alongside Circle-managed smart contracts that hold USDC reserves on source chains like Ethereum.

### Architecture

**Components:**
- Onchain smart contract (xReserve) on source chain
- Offchain attestation system (dual attesters)

**Domain System:**
- Source domains: small, 0-indexed integers (e.g., 0 = Ethereum)
- Remote domains: start at 10001

**Three Blockchain Roles:**
1. **Source chain**: Hosts xReserve contract and USDC reserves
2. **Remote chain**: Issues and circulates USDC-backed stablecoins
3. **Destination chain**: Where users ultimately withdraw funds

### Deposit Flow
1. User transfers USDC into xReserve contract on source blockchain
2. Smart contract locks funds and emits deposit event
3. Attestation service generates and signs deposit verification
4. Remote blockchain retrieves signed attestation
5. Remote token contract mints equivalent USDCx
6. Tokens transfer to user's remote blockchain wallet

### Withdrawal Flow
1. User burns USDCx on remote blockchain
2. Token contract burns stablecoins and emits burn event
3. Remote attester signs burn intent offchain
4. Burn intent + signature sent to xReserve
5. xReserve verifies burn and issues withdrawal attestation
6. System releases USDC to user on destination chain

**Advanced**: Withdrawal forwarding allows receiving USDC on a different chain than the source.

### DepositIntent Message Structure (240+ bytes)

| Field | Size | Description |
|---|---|---|
| magic | 4 bytes | `0x5a2e0acd` |
| version | 4 bytes | `1` |
| amount | 32 bytes | Token deposit amount |
| remoteDomain | 4 bytes | Destination domain ID |
| remoteToken | 32 bytes | Destination token address |
| remoteRecipient | 32 bytes | Destination recipient |
| localToken | 32 bytes | Source token address |
| localDepositor | 32 bytes | Source depositor |
| maxFee | 32 bytes | Maximum destination fee |
| nonce | 32 bytes | Replay protection |
| hookDataLen | 4 bytes | Hook payload length |
| hookData | variable | Optional extensibility |

### xReserve Contracts

**Mainnet (Ethereum):**
- USDC: `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`
- xReserve: `0x8888888199b2Df864bf678259607d6D5EBb4e3Ce`

**Testnet (Ethereum Sepolia):**
- USDC: `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238`
- xReserve: `0x008888878f94C0d87defdf0B07f46B93C1934442`

### Supported Remote Chains

| Chain | Domain ID | Mainnet Token Address | Testnet Token Address |
|---|---|---|---|
| Aleo | 10002 | `usdcx_stablecoin.aleo` | `test_usdcx_stablecoin.aleo` |
| Canton | 10001 | `InstrumentId.id: USDCx` | `InstrumentId.id: USDCx` |
| Cardano | 10004 | `1f3aec8bfe7ea4fe14c5f121e2a92e301afe414147860d557cac7e345553444378` | `31dde3db98ad05feb688d4dbb146b3b6054e1246cbcef98c79b0bf665553444378` |
| Stacks | 10003 | `SP120SBRBQJ00MCWS7TM5R8WJNTTKD5K0HFRC2CNE.usdcx` | `ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.usdcx` |

### USDC-Backed Stablecoin Specification

**Token Requirements:**
- 1:1 USDC collateral in xReserve contract
- 6 decimal places (matching native USDC)
- Use `uint256` for balances with overflow/underflow safeguards

**Required State Variables:**
- `uint32 domain` - Immutable blockchain identifier
- `mapping balances` - Account holdings (non-negative)
- `mapping usedNonces` - Deposit replay protection
- `mapping xReserveAttesters` - Approved signer allowlist
- `uint256 minBurnSize` - Minimum withdrawal amount
- `uint256 totalSupply` - Optional aggregate

**Mint Function**: Verifies deposit attestations using ECDSA recovery. Validates magic value `0x5a2e0acd`, version `1`, and nonce uniqueness.

**Burn Function**: Requires amount > 0, caller sufficiency, compliance with `minBurnSize`.

### xReserve Fees

| Fee Type | Amount |
|---|---|
| xReserve Protocol (Ethereum) | 0 bps |
| xReserve Protocol (other chains) | 1 bps |
| Gateway Protocol (supported chains) | 0.5 bps |
| Gas Fee (all destinations) | $2 USDC |
| Forwarding Fee | $0.20-$1.45 USDC |
| Forwarding Gas Fee | $0.001-$1.50 USDC |

Gateway-supported chains: Arbitrum, Avalanche, Base, HyperEVM, OP, Polygon PoS, Sei, Sonic, Unichain, World Chain.

### xReserve API Endpoints
- `GET /v1/attestations/{depositMessageHash}`
- `GET /v1/remote-domains/{remoteDomain}/attestations`
- `GET /v1/balances/{remoteDomain}`
- `GET /v1/info`
- `POST /v1/prepare-withdrawal`
- `POST /v1/withdraw`
- `GET /v1/withdrawal/{withdrawalId}`

### Deposit Tutorial (Ethereum Sepolia)

**Dependencies**: `viem`, `dotenv`

**Core Steps:**
1. Create `.env` with `PRIVATE_KEY` and chain-specific recipient address
2. Check ETH and USDC balances
3. Call `approve()` on USDC contract for xReserve
4. Call `depositToRemote()` on xReserve contract

**`depositToRemote()` parameters:**
- `value` (uint256) - Deposit amount
- `remoteDomain` (uint32) - Target chain ID
- `remoteRecipient` (bytes32) - Recipient (chain-specific encoding)
- `localToken` (address) - USDC contract address
- `maxFee` (uint256) - Max bridge fee
- `hookData` (bytes) - Recipient metadata

**Chain-specific encoding:**
- **Aleo (10002)**: Uses `@provablehq/sdk`, little-endian byte ordering via `Address.toBytesLe()`
- **Canton (10001)**: keccak256 hash of recipient address
- **Cardano (10004)**: Uses `@scure/base` for bech32 decoding, supports Base and Enterprise addresses
- **Stacks (10003)**: Uses `@stacks/transactions`, 11-byte padding + version + 20-byte hash

### Withdrawal Tutorial (Remote Blockchain)

**5-Step Process:**
1. **Monitor burn events** on remote chain (recipient address, remote domain ID)
2. **Prepare withdrawal** via `POST /prepare-withdrawal` with batch data
3. **Sign message hash** returned by prepare-withdrawal using attester keys (secp256k1)
4. **Submit withdrawal** via `POST /withdraw` with encoded payload + signatures
5. **Monitor status** via `GET /withdrawals/{withdrawalId}` (states: created -> verified -> finalized)

---

## 6. Stablecoin Transfers on Various Chains

### 6.1 Transfer USDC on EVM Chains

**Supported Testnet Chains & Addresses:**

| Chain | USDC Address |
|---|---|
| Arc Testnet | `0x3600000000000000000000000000000000000000` |
| Arbitrum Sepolia | `0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d` |
| Avalanche Fuji | `0x5425890298aed601595a70AB815c96711a31Bc65` |
| Base Sepolia | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| Celo Sepolia | `0x01C5C0122039549AD1493B8220cABEdD739BC44E` |
| Ethereum Sepolia | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` |
| Optimism Sepolia | `0x5fd84259d66Cd46123540766Be93DFE6D43130D7` |
| Polygon Amoy | `0x41E94Eb019C0762f9Bfcf9Fb1E58725BfB0e7582` |
| ZKsync Era Testnet | `0xAe045DE5638162fa134807Cb558E15A3F5A7F853` |

**Setup (TypeScript + Viem):**
```bash
mkdir transfer-usdc-evm && cd transfer-usdc-evm
npm init -y
npm pkg set type=module
npm pkg set scripts.start="npx tsx --env-file=.env index.ts"
npm install viem tsx
npm install --save-dev typescript @types/node
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

**.env:**
```
PRIVATE_KEY={your_private_key}
RECIPIENT_ADDRESS={recipient_address}
```

**Minimal ABI:**
- `balanceOf(address)` - Returns token balance
- `transfer(address, uint256)` - Transfers tokens

**Flow:** Check balance -> Validate sufficiency -> Parse amount -> Send transaction

### 6.2 Transfer EURC on EVM Chains

**Supported Testnet Chains:**

| Chain | EURC Address |
|---|---|
| Arc Testnet | `0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a` |
| Avalanche Fuji | `0x5E44db7996c682E92a960b65AC713a54AD815c6B` |
| Base Sepolia | `0x808456652fdb597867f38412077A9182bf77359F` |
| Ethereum Sepolia | `0x08210F9170F89Ab7658F0B5E3fF39b0E03C594D4` |
| World Chain Sepolia | `0xe479EcA5740Ac65d6E1823bea2f1C08Bc14e954F` |

Same setup as USDC EVM transfer (Viem + TypeScript). EURC also uses 6 decimals.

### 6.3 Transfer USDC on Solana

**USDC Mint Address (Devnet):** `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU`
**Decimals:** 6 (1 USDC = 1,000,000 units)
**RPC:** `https://api.devnet.solana.com`

**Dependencies:**
```bash
npm install @solana/kit @solana-program/token ws dotenv
```

**Core Steps:**
1. Store sender's private key as JSON byte array in `.env`
2. Derive Associated Token Account (ATA) addresses via `findAssociatedTokenPda()`
3. Create transfer instruction with SPL Token program
4. Sign and confirm with "confirmed" commitment level

**Transfer amount:** `1_000_000n` for 1 USDC

### 6.4 Transfer USDC on Aptos

**USDC Address (Testnet):** `0x69091fbab5f7d635ee7ac5098cf0c1efbe31d68fec0f2cd565e8d168daf52832`

**Dependencies:**
```bash
npm install @aptos-labs/ts-sdk @aptos-labs/wallet-adapter-ant-design @aptos-labs/wallet-adapter-react
```

**Transfer uses:**
- Function: `0x1::primary_fungible_store::transfer`
- Amount adjustment: multiply by 1,000,000 (6 decimals)
- Petra wallet adapter for signing

**Built with:** Next.js + TypeScript + Tailwind CSS

### 6.5 Transfer USDC on Starknet

**USDC Address (Starknet Sepolia):** `0x053b40A647CEDfca6cA84f542A0fe36736031905A9639a7f19A3C1e66bFd5080`
**Decimals:** 6

**Dependencies:**
```bash
npm install starknet@^8.9.0 dotenv
```

**Key Difference:** Starknet uses account abstraction by default - every wallet is a smart contract (not EOA). Requires both account address and private key.

**.env Variables:**
- `ACCOUNT_ADDRESS` (felt252 format, 0x prefix)
- `PRIVATE_KEY` (felt252 format, 0x prefix)
- `RECIPIENT_ADDRESS` (felt252 format, 0x prefix)

**ABI methods:** `balance_of`, `transfer`
**Balance conversion:** `uint256.uint256ToBN()`, divide by 10^6

### 6.6 Transfer USDC on Sui

**Contract Addresses:**
- **Testnet:** `0xa1ec7fc00a6f40db9693ad1415d0c193ad3906494428cf252621037bd7117e29::usdc::USDC`
- **Mainnet:** `0xdba34672e30cb065b1f93e3ab55318768fd6fef66c15942c9f7cb846e2f900e7::usdc::USDC`

**Dependencies:**
```bash
npm install @mysten/dapp-kit @mysten/sui @mysten/wallet-kit @tanstack/react-query
```

**Network endpoint:** `https://fullnode.testnet.sui.io:443`

**Transfer Steps:**
1. Retrieve USDC coins via `suiClient.getCoins()`
2. Create TransactionBlock
3. Split coins for transfer amount
4. Execute via `signAndExecuteTransactionBlock()`

**Verify on:** [Sui Explorer](https://suiscan.xyz/)

### 6.7 USDC Trustline on XRPL

**USDC Configuration (XRPL Testnet):**
- Currency: `5553444300000000000000000000000000000000`
- Issuer: `rHuGNhqTG32mfmAvWA8hUyWRLV3tCSwKQt`
- Limit value: `1000000000`

**WebSocket:** `wss://s.altnet.rippletest.net:51233`

**Dependencies:**
```bash
npm i xrpl typescript ts-node
```

**3-Step Process:**
1. **Generate wallet:** `Wallet.generate()` + `client.fundWallet()`
2. **Create trustline:** TrustSet transaction with `TrustSetFlags.tfSetNoRipple`
3. **Verify:** Check `response.result.validated === true` and `TransactionResult === "tesSUCCESS"`

---

## 7. Smart Contracts Platform

### Overview
Circle Contracts enables creating, deploying, and managing smart contracts through Developer Console or APIs. Supports both no-code and programmatic approaches.

### Supported Blockchains

**Mainnet:** Arbitrum, Avalanche, Base, Ethereum, Monad, OP Mainnet, Polygon PoS, Unichain

**Testnet:** Arbitrum Sepolia, Arc Testnet, Avalanche Fuji, Base Sepolia, Ethereum Sepolia, Monad Testnet, OP Sepolia, Polygon PoS Amoy, Unichain Sepolia

### Deployment via Bytecode (Custom Contracts)

**Prerequisites:**
- Circle Developer Account + API key
- Registered Entity Secret
- SDKs: `@circle-fin/developer-controlled-wallets` + `@circle-fin/smart-contract-platform`

**Process:**
1. Create developer-controlled wallet within a wallet set
2. Fund via Circle's Testnet Faucet
3. Compile contract (e.g., via Remix IDE) to get ABI + bytecode
4. Deploy via SDK's `deployContract()` method
5. Monitor via `getContract()` or Developer Console

**Example deployment (MerchantTreasuryUSDC contract):**
- Constructor params: owner address, USDC token address (`0x3600000000000000000000000000000000000000` on Arc Testnet)
- Fee level: "MEDIUM"
- Returns: `contractId` + `transactionId`

### Contract Interaction

**Read Operations (no gas):**
```
POST /contracts/query
```
Parameters: `abiFunctionSignature`, `address`, `blockchain`, `abiJson`

**Write Operations (requires gas + wallet):**
```
POST developer/transactions/contractExecution  (dev-controlled)
POST user/transactions/contractExecution       (user-controlled)
```
Parameters: `walletId`, `contractAddress`, `abiFunctionSignature`, `abiParameters`, `fee`

### Event Monitoring

**Setup:**
1. Configure HTTPS webhook endpoint for POST requests
2. Filter to `contracts.EventLog` notifications
3. Import contract via SDK (error 175001 if not imported)
4. Create event monitor with human-readable signature

**Import Contract:**
```javascript
const importResponse = await circleContractSdk.importContract({
  address: "0x6bc50ff08414717f000431558c0b585332c2a53d",
  blockchain: "ARB-SEPOLIA",
  idempotencyKey: "50d64a5e-6b2e-47ea-aa14-13feab9376e9",
  name: "MyToken",
  description: "My ERC-20 Token Contract",
});
```

**Create Event Monitor:**
```javascript
const monitorResponse = await circleContractSdk.createEventMonitor({
  blockchain: "ARB-SEPOLIA",
  contractAddress: "0x6bc50ff08414717f000431558c0b585332c2a53d",
  eventSignature: "Transfer(address,address,uint256)",
  idempotencyKey: "f80fcf44-bbb1-4336-870a-f1802ad98e0f",
});
```

**Webhook Notification Structure:**
- `subscriptionId`, `notificationId`
- `notificationType: "contracts.eventLog"`
- Contract address, blockchain, transaction hash
- Event name with parameter types
- Topics array (hashed indexed parameters)
- Raw data as hex string
- ISO 8601 timestamp

---

## 8. Contract Templates

### Available Templates

| Template | Standard | Template ID | Use Cases |
|---|---|---|---|
| Token | ERC-20 | `a1b74add-23e0-4712-88d1-6b3009e85a86` | Fungible tokens, loyalty points, governance |
| NFT | ERC-721 | `76b83278-50e2-4006-8b63-5b1a2a814533` | Digital collectibles, gaming assets |
| Multi-Token | ERC-1155 | `aea21da6-0aa2-4971-9a1a-5098842b1248` | Mixed token types, batch transfers |
| Airdrop | N/A | `13e322f2-18dc-4f57-8eed-4bddfc50f85e` | Bulk token distribution |

### ERC-20 Token Template

**Required Parameters:** `name`, `defaultAdmin`, `primarySaleRecipient`
**Optional:** `symbol`, `platformFeeRecipient`, `platformFeePercent`, `contractUri`, `trustedForwarders`

**Core Write Functions:**
- `approve(address spender, uint256 amount)` - Authorize spending
- `transfer(address to, uint256 amount)` - Send tokens
- `mintTo(address to, uint256 amount)` - Create tokens (requires MINTER_ROLE)
- `burn(uint256 amount)` - Destroy tokens
- `grantRole(bytes32 role, address account)` - Assign role
- `revokeRole(bytes32 role, address account)` - Remove role

**Read Functions:** `balanceOf(address)`, `allowance(address, address)`

### ERC-721 NFT Template

**Required Parameters:** `name`, `defaultAdmin`, `primarySaleRecipient`, `royaltyRecipient`, `royaltyPercent`
**Optional:** `symbol`, `platformFeeRecipient`, `platformFeePercent`, `contractUri`, `trustedForwarders`

**Core Functions:**
- `approve(address to, uint256 tokenId)` - Approve transfer
- `mintTo(address to, string uri)` - Mint NFT (requires MINTER_ROLE)
- `safeTransferFrom(address from, address to, uint256 tokenId, bytes data)` - Transfer with safety check
- `setTokenURI(uint256 tokenId, string uri)` - Update metadata
- `ownerOf(uint256 tokenId)` - Get owner
- `balanceOf(address owner)` - Get token count

### ERC-1155 Multi-Token Template

**Required Parameters:** `name`, `defaultAdmin`, `primarySaleRecipient`, `royaltyRecipient`, `royaltyPercent`

**Core Functions:**
- `mintTo(address, uint256, string, uint256)` - Mint tokens (MINTER_ROLE required)
  - For token ID 0, use max uint256: `115792089237316195423570985008687907853269984665640564039457584007913129639935`
- `safeTransferFrom(address, address, uint256, uint256, bytes)` - Transfer tokens
- `safeBatchTransferFrom(...)` - Batch transfer
- `setApprovalForAll(address, bool)` - Grant operator permissions
- `burn(address, uint256, uint256)` - Destroy tokens
- `balanceOf(address, uint256)` - Check balance
- `balanceOfBatch(address[], uint256[])` - Batch balance check
- `uri(uint256)` - Get metadata URI
- `nextTokenIdToMint()` - Next available ID

### Airdrop Template

**Required Parameter:** `defaultAdmin` (address)
**Optional:** `contractURI` (string)
**Max recipients per tx:** 500

**Core Functions:**
- `airdropERC20(address, tuple[])` - Distribute fungible tokens
- `airdropERC721(address, tuple[])` - Distribute NFTs
- `airdropERC1155(address, tuple[])` - Distribute multi-tokens
- `setOwner(address)` - Transfer ownership

### Template Deployment via API

**Deploy Endpoint:** `POST /templates/{templateId}/deploy`

**Node.js Example (ERC-1155):**
```javascript
const response = await circleContractSdk.deployContractTemplate({
  id: "aea21da6-0aa2-4971-9a1a-5098842b1248",
  blockchain: "ARC-TESTNET",
  name: "MyERC1155Contract",
  walletId: "<WALLET_ID>",
  templateParameters: {
    name: "MyERC1155Contract",
    defaultAdmin: "<WALLET_ADDRESS>",
    primarySaleRecipient: "<WALLET_ADDRESS>",
    royaltyRecipient: "<WALLET_ADDRESS>",
    royaltyPercent: 0,
  },
  fee: { type: "level", config: { feeLevel: "MEDIUM" } },
});
```

**cURL Example:**
```bash
curl --request POST \
  --url 'https://api.circle.com/v1/w3s/templates/aea21da6-0aa2-4971-9a1a-5098842b1248/deploy' \
  --header 'authorization: Bearer <API_KEY>' \
  --header 'content-type: application/json' \
  --data '{
    "idempotencyKey": "<IDEMPOTENCY_KEY>",
    "blockchain": "ARC-TESTNET",
    "name": "MyERC1155Contract",
    "walletId": "<WALLET_ID>",
    "templateParameters": {
      "name": "MyERC1155Contract",
      "defaultAdmin": "<WALLET_ADDRESS>",
      "primarySaleRecipient": "<WALLET_ADDRESS>",
      "royaltyRecipient": "<WALLET_ADDRESS>",
      "royaltyPercent": 0
    },
    "feeLevel": "MEDIUM",
    "entitySecretCiphertext": "<ENTITY_SECRET_CIPHERTEXT>"
  }'
```

**Mint Token (ERC-1155):**
```javascript
const response = await circleDeveloperSdk.createContractExecutionTransaction({
  walletId: "<WALLET_ID>",
  abiFunctionSignature: "mintTo(address,uint256,string,uint256)",
  abiParameters: [
    "<WALLET_ADDRESS>",
    "115792089237316195423570985008687907853269984665640564039457584007913129639935",
    "ipfs://bafkreibdi6623n3xpf7ymk62ckb4bo75o3qemwkpfvp5i25j66itxvsoei",
    "1",
  ],
  contractAddress: "<CONTRACT_ADDRESS>",
  fee: { type: "level", config: { feeLevel: "MEDIUM" } },
});
```

---

## 9. SDKs

### SDK Overview

| SDK | Package (npm/pip) | Languages |
|---|---|---|
| Bridge Kit | `@circle-fin/bridge-kit` | TypeScript |
| Dev-Controlled Wallets | `@circle-fin/developer-controlled-wallets` / `circle-developer-controlled-wallets` | TypeScript, Python |
| Modular Wallets | `@circle-fin/modular-wallets-core` | JS, TS, Swift, Kotlin, Java |
| User-Controlled Wallets (Client) | GitHub repos | JS, TS, Kotlin, Swift |
| User-Controlled Wallets (Server) | `@circle-fin/user-controlled-wallets` / `circle-user-controlled-wallets` | TypeScript, Python |
| Contracts | `@circle-fin/smart-contract-platform` / `circle-smart-contract-platform` | TypeScript, Python |
| Mint Payouts | GitHub `circlefin/circle-nodejs-sdk` | Node.js |

### Contracts Node.js SDK

**Install:**
```bash
npm install @circle-fin/smart-contract-platform --save
```

**Prerequisites:** Node.js v22+, API key (Standard Key type), Registered Entity Secret

**Initialization:**
```javascript
import { initiateSmartContractPlatformClient } from "@circle-fin/smart-contract-platform";
const client = initiateSmartContractPlatformClient({
  apiKey: "<your-api-key>",
  entitySecret: "<your-entity-secret>",
});
```

**Config options:** `apiKey` (required), `entitySecret` (required), `storage` (optional, defaults to in-memory)

### Contracts Python SDK

**Install:**
```bash
pip install circle-smart-contract-platform
```

**Initialization:**
```python
from circle.web3 import utils
client = utils.init_smart_contract_platform_client(
  api_key="Your API KEY",
  entity_secret="Your entity secret"
)
```

**Import contract example:**
```python
from circle.web3 import smart_contract_platform
api_instance = smart_contract_platform.DeployImportApi(client)
request = smart_contract_platform.ImportContractRequest.from_dict({
    "name": "UChildERC20Proxy",
    "address": "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
    "blockchain": "MATIC"
})
response = api_instance.import_contract(request)
```

**Optional params:** `host` (custom base URL, default `https://api.circle.com/v1/w3s`), `user_agent`

### Pydantic v2 Upgrade (Python SDKs)

Starting with version 9.0.0, Circle Python SDKs upgrade to Pydantic v2.

**Breaking changes** - Model serialization method naming:
| Method | Purpose |
|---|---|
| `model.from_dict()` | Deserialize dict to model |
| `model.from_json()` | Deserialize JSON to model |
| `model.to_dict()` | Serialize model to dict |
| `model.to_json()` | Serialize model to JSON |

**Migration example:**
```python
wallet_dict = wallet_model.to_dict()
wallet_json = wallet_model.to_json()
wallet_from_dict = EOAWallet.from_dict(wallet_dict)
wallet_from_json = EOAWallet.from_json(wallet_json)
```

Version 8 enters maintenance mode (critical security patches only).

---

## 10. AI / MCP Integration

### AI Chatbot (codegen.circle.com)
- AI-assisted development environment at https://codegen.circle.com
- Supports code generation for: Circle Wallets, Contracts, CCTP, Gateway
- Uses React iframe component (210px default, expands to 600px with messages)
- Communicates via postMessage API with origin validation

### MCP Server

**Server Configuration:**
- Name: `circle`
- URL: `https://api.circle.com/v1/codegen/mcp`

**Quick CLI Installation:**
```bash
# Claude Code
claude mcp add --transport http circle https://api.circle.com/v1/codegen/mcp --scope user

# Codex
codex mcp add circle --url https://api.circle.com/v1/codegen/mcp
```

**IDE-Specific Setup:**

**Cursor:** Add via MCP settings with server URL, enable via toggle.

**Windsurf:** Configure in `~/.codeium/windsurf/mcp_config.json`, then enable in MCP settings.

**Kiro:** Workspace-level (`./.kiro/settings/mcp.json`) or user-level config. Uses `npx @circle/mcp-server` with `CIRCLE_BASE_URL` env var.

---

## 11. Cross-Chain Transfers

### Product Comparison

| Product | Description | Best For |
|---|---|---|
| **Bridge Kit** | SDK for USDC transfers across EVM and non-EVM chains | Simple cross-chain transfers, minimal setup |
| **CCTP** | Permissionless burn-and-mint protocol | Smart contract integration, composable flows, speed-sensitive |
| **Gateway** | Unified balance across chains | Multichain apps, chain abstraction |
| **Nanopayments** | Gas-free sub-cent USDC payments | Machine-to-machine, AI agents, usage-based billing |

### CCTP Features
- Fast Transfer (speed-optimized) and Standard Transfer options
- Automated actions post-transfer using Hooks
- V2 deployed on 25+ chains as of early 2026

### Nanopayments
- Minimum payment: $0.000001
- Batched settlement
- Gas-free for users

---

## 12. Liquidity Services

### Circle Mint
- Direct USDC/EURC minting and redemption
- Automatic fiat-to-stablecoin conversion from bank accounts
- 1:1 redemption guarantee
- Web UI and API integration
- Instant settlement via participating banking partners
- 24/7 global distribution
- Target users: exchanges, wallet providers, custodians, banks, PSPs

### StableFX
- Institutional FX trading engine
- Request-for-Quote (RFQ) execution with onchain settlement
- Multi-provider liquidity aggregation via single API call
- Arc blockchain-based trading with sub-second finality
- Atomic smart contract settlement
- 24/7 trading availability
- Supports USDC <-> EURC conversions
- Target users: payment providers, fintechs, OTC desks, prime brokers

---

## 13. Payments Network

### Circle Payments Network (CPN)
- Institutional cross-border payment infrastructure
- Near-instant settlement via USDC
- Integrated compliance (end-to-end encryption, travel rule)

### 4-Stage Workflow
1. **Quote Generation** - FX quotes from multiple providers, rate locks
2. **Payment Creation** - OFI submits with encrypted compliance data, BFI reviews/approves
3. **Onchain Transaction** - USDC transferred via blockchain with confirmation notifications
4. **Fiat Settlement** - BFI converts USDC to local currency, real-time status updates

### Prerequisites for OFIs
- USDC liquidity access
- Custodial solution + blockchain interaction capability
- Established KYC/AML processes

---

## 14. Sample Projects

### Wallets Projects
| Project | Stack | Description |
|---|---|---|
| AI-Powered Escrow | TypeScript/Web | AI agent + blockchain for gig economy escrow |
| Autonomous Payments | Python | USDC payments with AI agents using dev-controlled wallets |
| Smart Account & Gasless | JS/React/Web | Passkey auth, user operations on Polygon Amoy |
| Multi-Platform Auth | JS/TS/Kotlin/Swift | iOS, Android, Web, React Native login flows |
| iOS Social Login | TypeScript/iOS | Social login sample |
| Session Management Server | JavaScript | Test server for user-controlled wallets |
| Telegram Bot | JavaScript | Dev-controlled wallets + USDC on Telegram |

### Paymaster
| Project | Stack | Description |
|---|---|---|
| USDC Gas Fee Payment | TypeScript/Web | Circle Paymaster to pay gas in USDC |

### Circle Mint
| Project | Stack | Description |
|---|---|---|
| Payment Flow Testing | Vue.js/Web | Mint Payments API demo |

### CCTP
| Project | Stack | Description |
|---|---|---|
| Cross-Chain USDC Transfers | TypeScript/React | CCTP fast transfer demo |
| Telegram Cross-Chain Bot | JavaScript | Dev wallets + CCTP on Telegram |

### Circle Research (Experimental)
| Project | Stack | Description |
|---|---|---|
| Ethereum Confirmation Rules | Python | Fast confirmation rule evaluation |
| TXT2TXN | JS + Python | Intent-based onchain transactions with AI |
| Perimeter Protocol | TypeScript | Credit apps using USDC |

---

## 15. Release Notes

### 2026 Highlights

**USDC (2026):**
- Feb 26: Morph Hoodi Testnet support
- Feb 09: EDGE Testnet support
- Jan 23: EVM transfer quickstarts consolidated

**EURC (2026):**
- Jan 23: Transfer guide expanded to all supported EVM chains

**xReserve (2026):**
- Feb 27: Cardano mainnet + preprod testnet support (USDCx)
- Jan 27: Aleo mainnet + testnet support (USDCx)
- Jan 15: USDC-backed Stablecoin Specification published

### 2025 Key Milestones

**New Chain Support:**
- Nov 24: USDC + CCTP V2 on Monad mainnet
- Oct 27: USDC, EURC, CCTP V2 on Arc testnet
- Sep 16: USDC + CCTP V2 on HyperEVM
- Sep 04: USDC on Plume
- Aug 26: USDC + CCTP V2 on XDC
- Jul 10: USDC on Sei
- Jun 12: USDC on XRPL
- Jun 03: USDC bridge-to-native on World Chain
- May 13: USDC bridge-to-native on Sonic
- Mar 17: USDC on Linea
- Jan 07: USDC on Aptos

**CCTP V2 Rollout (2025):**
- Mar 11: V2 launched on Avalanche, Base, Ethereum (mainnet)
- Jun 10-26: Solana, OP, Polygon PoS, Unichain
- Sep-Nov: HyperEVM, Plume, XDC, Ink, Starknet, Monad

**xReserve (2025):**
- Dec 18: USDCx on Stacks
- Dec 04: xReserve launched with Canton
- Nov 18: xReserve announced

**Tokenized Funds (2025):**
- Jun 11: USYC smart contract addresses published
- May 28: USYC overview, subscribe/redeem quickstart, Web2 APIs launched

**Other 2025:**
- Jul 23: USYC cross-chain transfers (ETH <-> BNB) via CCTP V2
- Jun 10: New fee endpoint: `GET /v2/burn/USDC/fees/:sourceDomainId/:destDomainId`
- Mar 27: Renamed to "CCTP" (from "Cross-Chain Transfer Protocol")
- Jan 23: Paymaster launched
- Jan 09: CCTP V1 on Aptos

### 2024 Key Milestones
- Dec 17: CCTP V1 on Sui
- Oct 08: USDC transfer quickstart on Sui
- Sep 17: USDC on Sui mainnet + testnet

---

## Quick Reference: Key Contract Addresses (Arc Testnet)

| Asset | Address |
|---|---|
| USDC | `0x3600000000000000000000000000000000000000` |
| EURC | `0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a` |

## Quick Reference: Template IDs

| Template | ID |
|---|---|
| ERC-20 Token | `a1b74add-23e0-4712-88d1-6b3009e85a86` |
| ERC-721 NFT | `76b83278-50e2-4006-8b63-5b1a2a814533` |
| ERC-1155 Multi-Token | `aea21da6-0aa2-4971-9a1a-5098842b1248` |
| Airdrop | `13e322f2-18dc-4f57-8eed-4bddfc50f85e` |

## Quick Reference: API Base URLs

| Service | URL |
|---|---|
| Circle API | `https://api.circle.com/v1/w3s/` |
| MCP Server | `https://api.circle.com/v1/codegen/mcp` |
| Gateway Testnet | `https://gateway-api-testnet.circle.com/v1/info` |
| IRIS Sandbox | `https://iris-api-sandbox.circle.com/v2/burn/USDC/fees/` |
| Docs Index | `https://developers.circle.com/llms.txt` |
