# Circle & Arc Documentation Summary

A comprehensive, structured summary of **Circle platform** and **Arc Network** official documentation -- condensed from **606 pages** into **9 well-organized Markdown files** (~11,500 lines total).

All information is sourced and consolidated from the official [Circle](https://developers.circle.com/) and [Arc](https://docs.arc.network) websites, structured into clean Markdown files so that AI assistants (Claude, Cursor, ChatGPT, etc.) can directly ingest and reference the full technical details in a single context.

## What is Circle & Arc?

- **Circle** is a global financial technology company and the issuer of **USDC**. It provides a full platform of services: wallets, cross-chain transfers (CCTP), payment networks (CPN), compliance tools, and more.
- **Arc** is Circle's own **Layer 1 blockchain** -- purpose-built for USDC-native settlement with sub-second finality, EVM compatibility, and built-in privacy features.

## File Index

### Circle Documentation (582 pages summarized)

| File | Content | Lines |
|------|---------|-------|
| `circle-api-cctp.md` | CCTP API (8 endpoints) - Cross-chain USDC transfers | 508 |
| `circle-api-mint.md` | Circle Mint API (63 endpoints) - Fiat on/off ramp | 1,626 |
| `circle-api-w3s.md` | W3S / Wallets / Smart Contracts API (111 endpoints) | 995 |
| `circle-api-gateway-cpn-stablefx.md` | Gateway / CPN / StableFX / xReserve API (65 endpoints) | 1,504 |
| `circle-wallets.md` | Wallet guides: Developer-Controlled, User-Controlled, Modular, Gas Station | 1,227 |
| `circle-crosschain.md` | Cross-chain: CCTP, Bridge Kit, Gateway, Nanopayments, x402 | 1,065 |
| `circle-services.md` | Services: Mint, CPN, StableFX, Compliance, Paymaster | 1,844 |
| `circle-assets-sdk-misc.md` | Assets (USDC/EURC), SDKs, AI-MCP, Sample Apps, Release Notes | 1,139 |

### Arc Network Documentation (24 pages summarized)

| File | Content | Lines |
|------|---------|-------|
| `arc-network-docs.md` | Concepts (consensus, execution, finality, privacy, fees) + Reference (RPC, contracts, EVM differences) + Tools + Tutorials | 1,636 |

## Key Topics Covered

- **247 API endpoints** fully documented with methods, paths, parameters, and response fields
- **CCTP v2** cross-chain transfer protocol with hooks support
- **Programmable Wallets** (Developer-Controlled, User-Controlled, Modular)
- **Smart Contract Accounts (SCA)** with ERC-4337 + ERC-6900
- **Bridge Kit** for USDC bridging between chains
- **Gateway** for institutional USDC minting/redemption
- **Nanopayments & x402** for micropayments via HTTP 402
- **CPN** (Circle Payments Network) for real-time institutional settlement
- **Arc Network** architecture: HotStuff consensus, parallel EVM, sub-second finality, confidential transactions

## Usage

These files are plain Markdown -- use them as:
- Quick reference when building with Circle APIs
- Context files for AI coding assistants (Claude, Cursor, etc.)
- Offline documentation for Arc and Circle development

## Disclaimer

This is an **unofficial, community-created summary** based on Circle's public documentation. For the most up-to-date and authoritative information, always refer to the official sources:

- Circle Docs: https://developers.circle.com/
- Arc Network Docs: https://docs.arc.network/

## License

This summary is provided for educational and development reference purposes. Circle and Arc trademarks belong to their respective owners.
