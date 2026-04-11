# Circle & Arc 技术更新（2026 年 4 月）

> 本文档补充 606 页文档总结，记录 2026 年 3-4 月的最新技术动态。

## Arc Network 最新进展

### 1. 开源 + Bug Bounty（2026-04-09）
- Arc Testnet 代码完全开源: https://github.com/circlefin/arc-node
- Malachite 共识引擎开源: https://github.com/circlefin/malachite
- HackerOne Bug Bounty 上线: https://hackerone.com/circle-bbp
- 开发者可运行自己的 Arc 节点（Full Node，非 Validator）

### 2. 后量子密码学（Post-Quantum Cryptography）
- Arc 将在主网上线时支持抗量子攻击功能
- 实现方案: **CRYSTALS-Dilithium (ML-DSA)** + **Falcon**（NIST 2024 标准化）
- 预编译地址: `0x1800000000000000000000000000000000000004`（PQ Signature Verify）
- **分 4 阶段推出**:
  - Phase 1（主网）: opt-in 抗量子钱包和签名
  - Phase 2: 私有状态加密（保护余额和交易数据）
  - Phase 3: Validator 层量子保护
  - Phase 4: 链下基础设施（通信、云、HSM、访问控制）
- 参考: SLH-DSA-SHA2-128s 预编译已在 testnet 代码中实现

### 3. Arc 技术栈确认
- **共识层**: Malachite BFT（Tendermint 变体，由 Informal Systems 开发，现归 Circle 维护）
- **执行层**: 基于 Reth（Paradigm 的 Rust EVM 客户端）
- **P2P**: libp2p + gossipsub
- **出块**: 亚秒级确定性终局
- **Gas**: USDC 作为原生 Gas 代币（linked interface，18 decimals native / 6 decimals ERC20）

### 4. 主网路线图
- 2025 年底: Public Testnet 上线
- 2026 年初: 开源代码 + Bug Bounty
- 2026 年中: 主网 Beta
- 2026 年: 全面主网上线

## CCTP 更新

### CCTP V2 成为唯一官方版本
- CCTP V1 于 2026-07-31 开始手动退出
- V2 已覆盖 **17+ 条链**
- 新增链: World Chain、Stellar（计划中）、Aptos + Sui（2026 上半年）

### CCTP 资产扩展
- 2026 年将支持更多资产的 burn-and-mint:
  - **EURC**（欧元稳定币）
  - **USYC**（Hashnote 基金代币化）
  - **cirBTC**
- 开放给第三方资产发行方

### 迁移
- V1 → V2 迁移指南: https://developers.circle.com/cctp/migration-from-v1-to-v2
- Domain ID: ARC = 26

## Modular Wallets 更新

### 当前状态
- 基于 ERC-6900 + ERC-4337 标准
- 支持链: Polygon PoS, Arbitrum
- SDK: `@circle-fin/modular-wallets-core`

### 即将推出的模块
- Session Keys
- Multi-Owner
- Weighted Multi-Sig

### 功能
- Circle Paymaster（Gas 抽象）
- 实时交易索引
- OFAC 制裁名单自动筛查
- Passkey 支持

## AI 开发者工具

### Circle Docs AI-Native
- 所有 Circle Docs 页面支持 `.md` 后缀获取结构化 Markdown
- AI agent 专用入口点
- 与 Anthropic Claude Agent SDK 集成
- Circle Skills: AI 工具帮助开发者更快集成

## 未来功能预告

- **Enshrined Paymasters**: EURC 和其他稳定币作为 Gas
- **原生 FX 功能**: 链上外汇
- **可配置隐私**: 保密余额和交易（opt-in）
- **Circle Payments Network (CPN)**: 扩展更多货币和简化接入

## 参考链接

- Arc 官网: https://www.arc.network/
- Arc 文档: https://docs.arc.network/
- Arc 社区: https://community.arc.network/
- Circle 开发者: https://developers.circle.com/
- Circle 2026 产品愿景: https://www.circle.com/blog/building-the-internet-financial-system-circles-product-vision-for-2026
- Arc 开源博客: https://www.arc.network/blog/open-sourcing-arc-run-your-own-arc-node-and-bug-bounty-program
- CCTP V2 迁移: https://developers.circle.com/cctp/migration-from-v1-to-v2
