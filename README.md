# Circle & Arc Documentation Summary

A comprehensive, structured summary of **Circle platform** and **Arc Network** official documentation -- condensed from **~636 pages** into **9 well-organized Markdown files** (~12,600 lines total).

All information is sourced and consolidated from the official [Circle](https://developers.circle.com/) and [Arc](https://docs.arc.network) websites, structured into clean Markdown files so that AI assistants (Claude, Cursor, ChatGPT, etc.) can directly ingest and reference the full technical details in a single context.

## What is Circle & Arc?

- **Circle** is a global financial technology company and the issuer of **USDC**. It provides a full platform of services: wallets, cross-chain transfers (CCTP), payment networks (CPN), compliance tools, and more.
- **Arc** is Circle's own **Layer 1 blockchain** -- purpose-built for USDC-native settlement with sub-second finality, EVM compatibility, and built-in privacy features.

## File Index

### Circle Documentation (~590 pages summarized)

| File | Content | Lines |
|------|---------|-------|
| `circle-api-cctp.md` | CCTP API (8 endpoints) - Cross-chain USDC transfers | 508 |
| `circle-api-mint.md` | Circle Mint API (72 endpoints, incl. Credit API) - Fiat on/off ramp + credit | 1,705 |
| `circle-api-w3s.md` | W3S / Wallets / Smart Contracts API (111 endpoints) | 995 |
| `circle-api-gateway-cpn-stablefx.md` | Gateway / CPN / StableFX / xReserve API (65 endpoints) | 1,504 |
| `circle-wallets.md` | Wallet guides: Developer-Controlled, User-Controlled, Modular, Gas Station | 1,229 |
| `circle-crosschain.md` | Cross-chain: CCTP, App Kit, Gateway, Nanopayments, x402 | 1,216 |
| `circle-services.md` | Services: Mint, CPN, StableFX, Compliance, Paymaster | 1,857 |
| `circle-assets-sdk-misc.md` | Assets (USDC/EURC), SDKs, AI-MCP, Circle Skills, Sample Apps, Release Notes | 1,172 |

### Arc Network Documentation (50 pages summarized)

| File | Content | Lines |
|------|---------|-------|
| `arc-network-docs.md` | Concepts (8) + References (5) + Tools (5) + Tutorials (9) + App Kit (25 pages: Bridge/Swap/Send SDK) | 2,344 |

## Key Topics Covered

- **256 API endpoints** fully documented with methods, paths, parameters, and response fields
- **CCTP v2** cross-chain transfer protocol with hooks support (24+ chains, V1 deprecating July 2026)
- **Programmable Wallets** (Developer-Controlled, User-Controlled, Modular)
- **Smart Contract Accounts (SCA)** with ERC-4337 + ERC-6900
- **App Kit** (`@circle-fin/app-kit`) for Bridge, Swap, and Send across 20+ chains
- **Gateway** for institutional USDC minting/redemption
- **Nanopayments & x402** for micropayments via HTTP 402
- **CPN** (Circle Payments Network) for real-time institutional settlement
- **Credit API** for Settlement Advance and Line of Credit products
- **Circle Skills** — 8 open-source AI skills for Claude Code, Cursor, Codex
- **Arc Network** architecture: Malachite BFT consensus, Reth execution, sub-second finality, post-quantum roadmap
- **ERC-8004** AI Agent identity + **ERC-8183** Agentic Commerce job lifecycle

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
