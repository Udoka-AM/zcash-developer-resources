# Zcash Developer Resources

> **An opinionated, engineering-first guide to building private applications, wallets, and zero-knowledge infrastructure on Zcash.**

This repository curates verified documentation, Rust/TypeScript/Mobile SDKs, zero-knowledge proving libraries (Halo 2), node & indexer infrastructure, threshold signature tools (FROST), and grant pathways for developers building on Zcash.

Unlike general community wikis, this index prioritizes **production-ready, actively maintained tooling**—highlighting the ecosystem's transition from legacy C++ infrastructure (`zcashd`) to the modern modular Rust stack (`Zebra` + `Zaino` + `Zallet` + `librustzcash`).

---

## Table of Contents

- [How Zcash Works (Architecture & Cryptography)](#how-zcash-works-architecture--cryptography)
- [Core Specifications & ZIPs](#core-specifications--zips)
- [Modern vs. Legacy Stack (Start Here)](#modern-vs-legacy-stack-start-here)
- [Core GitHub Repositories](#core-github-repositories)
- [SDKs, Libraries & Client Frameworks](#sdks-libraries--client-frameworks)
- [Node Infrastructure, Indexers & RPC Providers](#node-infrastructure-indexers--rpc-providers)
- [Zero-Knowledge & Cryptography Tooling (Halo 2 & FROST)](#zero-knowledge--cryptography-tooling-halo-2--frost)
- [Block Explorers & Network Monitoring](#block-explorers--network-monitoring)
- [Security, Audits & Formal Verification](#security-audits--formal-verification)
- [Video Talks, Workshops & Conferences](#video-talks-workshops--conferences)
- [Grants, Funding & Bounties](#grants-funding--bounties)
- [Communities & Working Groups](#communities--working-groups)
- [Contributing](#contributing)

---

## How Zcash Works (Architecture & Cryptography)

*Core conceptual resources explaining shielded pools (Sapling & Orchard), Unified Addresses, zero-knowledge proofs, and consensus.*

- [Zcash Protocol Specification (NU7)](https://zips.z.cash/protocol/protocol.pdf) - The canonical mathematical and cryptographic specification governing consensus, Orchard, Sapling, and transaction formats.
- [Zcash Official Documentation](https://zcash.readthedocs.io/en/latest/rtd_pages/basics.html) - Foundational overview of shielded transactions, viewing keys, memo fields, and zk-SNARKs.
- [Zcash Core DAG & Ecosystem Map](https://zcash.github.io/developers/zcash-core-dag) - Dependency graph mapping how core Zcash crates, wallets, and consensus components fit together.
- [Zcash GitHub Engineering Portal](https://zcash.github.io/) - Central hub for `Zcashd`, `Zallet`, cryptographic library docs, and active development roadmaps.
- [librustzcash Developer Onboarding Guide](https://dannywillems.github.io/librustzcash/) - Architectural walkthrough of Zcash's core Rust crates (`sapling`, `orchard`, `zcash_primitives`, `zcash_client_backend`).
- [ZecHub Wiki](https://github.com/ZecHub/zechub) - Open-source community education hub covering ecosystem history, tutorials, and contributor guides.

---

## Core Specifications & ZIPs

*Zcash Improvement Proposals (ZIPs) define every protocol upgrade, address standard, and wallet behavior.*

- [Zcash Improvement Proposals (ZIPs) Repository](https://github.com/zcash/zips) - Source repository for all draft, proposed, and active ZIPs.
- **Essential ZIPs Every Builder Should Read:**
  - **ZIP-32:** Shielded Hierarchical Deterministic Wallets
  - **ZIP-316:** Unified Addresses & Unified Viewing Keys (UAs / UVKs)
  - **ZIP-321:** Payment Request URIs for seamless wallet checkout flows
  - **ZIP-224 / ZIP-225:** Orchard Shielded Protocol & v5 Transaction Format
  - **ZIP-244:** Transaction Identifier Non-Malleability

---

## Modern vs. Legacy Stack (Start Here)

*If you are starting a new Zcash integration, build on the modular Rust stack rather than legacy C++ components.*

| Layer | Modern Production Stack (Recommended) | Legacy Stack (Maintenance / Deprecation) |
| :--- | :--- | :--- |
| **Consensus Node** | [Zebra](https://github.com/ZcashFoundation/zebra) (Rust) | [zcashd](https://github.com/zcash/zcash) (C++) |
| **Wallet Backend** | [Zallet](https://github.com/zcash/wallet) / [librustzcash](https://github.com/zcash/librustzcash) | `zcashd` built-in wallet |
| **Light Client Indexer** | [Zaino](https://github.com/zingolabs/zaino) (Rust) | [lightwalletd](https://github.com/zcash/lightwalletd) (Go) |
| **Browser / Web SDK** | [WebZjs](https://github.com/ChainSafe/WebZjs) (Wasm + TypeScript) | Custom RPC wrappers |
| **Multi-Sig / Custody** | [FROST](https://github.com/ZcashFoundation/frost) (Threshold Schnorr) | Transparent P2SH multisig |

---

## Core GitHub Repositories

*Primary protocol implementations maintained by the Zcash Foundation, Electric Coin Company (ECC), Shielded Labs, and ZingoLabs.*

- [ZcashFoundation/zebra](https://github.com/ZcashFoundation/zebra) - Independent, modular Rust implementation of a Zcash consensus node.
- [zcash/librustzcash](https://github.com/zcash/librustzcash) - Core Rust monorepo containing `zcash_primitives`, `zcash_proofs`, `zcash_client_backend`, and `zcash_client_sqlite`.
- [zcash/halo2](https://github.com/zcash/halo2) - Zero-knowledge proving system using PLONKish arithmetization and inner-product arguments without a trusted setup.
- [zingolabs/zaino](https://github.com/zingolabs/zaino) - High-performance Rust blockchain indexer replacing legacy `lightwalletd` infrastructure.
- [zcash/zcash](https://github.com/zcash/zcash) - Original C++ consensus node and protocol reference implementation.
- [zcash/lightwalletd](https://github.com/zcash/lightwalletd) - Go backend gRPC service providing bandwidth-efficient compact blocks to mobile/light wallets.
- [ShieldedLabs/zebra-crosslink](https://github.com/ShieldedLabs) - Hybrid Proof-of-Work / Proof-of-Stake (Crosslink) protocol R&D and Zebra prototype.
- [QED-it/zcash_tx_tool](https://github.com/QED-it/zcash_tx_tool) - Transaction generation and testing utility supporting v5/v6 transactions and Zcash Shielded Assets (ZSAs).

---

## SDKs, Libraries & Client Frameworks

### Web & JavaScript / TypeScript
- [ChainSafe/WebZjs](https://github.com/ChainSafe/WebZjs) - Browser-native JavaScript/TypeScript SDK compiled from Rust/Wasm for shielded Zcash transactions in web apps.
- [@chainsafe/webzjs-zcash-snap](https://www.npmjs.com/package/@chainsafe/webzjs-zcash-snap) - MetaMask Snap integration enabling shielded Zcash support inside browser wallets.

### Rust Wallet & Application APIs
- [zingolabs/zingolib](https://github.com/zingolabs/zingolib) - Full-featured Rust wallet library and CLI test harness exposing shielded sync, memos, and Unified Address management.
- [zingolabs/zingolib-crosslink](https://github.com/zingolabs) - Experimental fork of `zingolib` built for testing Crosslink hybrid consensus features.

### Mobile SDKs (iOS & Android)
- [zcash/zcash-swift-wallet-sdk](https://github.com/zcash/zcash-swift-wallet-sdk) - Native Swift light-client SDK for iOS wallets backed by `librustzcash` FFI bindings.
- [zcash/zcash-android-wallet-sdk](https://github.com/zcash/zcash-android-wallet-sdk) - Kotlin/Java light-client SDK for Android (note: check NU7 compatibility roadmap before new deployments).

---

## Node Infrastructure, Indexers & RPC Providers

*Run your own sovereign infrastructure or connect to managed RPC endpoints.*

### Self-Hosted Node & Indexer Tooling
- [Zebra Full Node](https://github.com/ZcashFoundation/zebra) - Production Rust full node with optional block pruning and JSON-RPC support.
- [Zaino Rust Indexer](https://github.com/zingolabs/zaino) - Rust drop-in indexer exposing `CompactTxStreamer` gRPC interfaces for light wallets.
- [Lightwalletd Configuration Guide](https://github.com/gesker/lightwalletd_config) - Step-by-step setup guide for pairing `lightwalletd` with a node on mainnet and testnet.

### Managed RPC Providers
- [Tatum Zcash RPC](https://openchainbench.com/benchmarks) - High-availability Zcash RPC gateway with free-tier developer endpoints.
- [NOWNodes Zcash API](https://chainstack.com/learn/compare/best-zcash-rpc-providers/) - Managed Zcash mainnet JSON-RPC, Blockbook REST API, and WebSocket endpoints.
- [GetBlock Zcash Nodes](https://chainstack.com/learn/compare/best-zcash-rpc-providers/) - Dedicated and shared JSON-RPC node access for Zcash mainnet and testnet.

---

## Zero-Knowledge & Cryptography Tooling (Halo 2 & FROST)

*Zcash's cryptographic primitives are used across the broader ZK ecosystem—build circuits or threshold signing flows directly with them.*

- [The Halo 2 Book](https://zcash.github.io/halo2/user/simple-example.html) - Official developer book on writing arithmetic circuits, custom gates, and lookup arguments in `halo2`.
- [halo2_gadgets](https://github.com/zcash/halo2/blob/main/halo2_gadgets/README.md) - Reusable circuit gadgets including Poseidon hash, Sinsemilla, ECC operations, and SHA-256.
- [ZcashFoundation/frost](https://github.com/ZcashFoundation/frost) - Rust implementation of Flexible Round-Optimised Schnorr Threshold (FROST) signatures for decentralized multi-party custody.
- [ZcashFoundation/frost-tools](https://github.com/ZcashFoundation/frost-tools) - CLI and signing utilities for constructing and signing Zcash transactions with external FROST threshold signatures.

---

## Block Explorers & Network Monitoring

*Inspect transparent transactions, shielded pool metrics, and network health.*

- [ZecBlock (formerly CipherScan)](https://github.com/Kenbak/zecblock) - Open-source, privacy-focused Zcash block explorer tracking blocks, transactions, and shielded pool balances.
- [Blockchair Zcash Explorer](https://blockchair.com/zcash) - Multi-chain block explorer with SQL-like filtering and mempool analytics for Zcash.
- [3xpl Zcash Explorer](https://3xpl.com) - Fast, ad-free universal block explorer supporting Zcash network data.

---

## Security, Audits & Formal Verification

*Security advisories, vulnerability reporting, and cryptographic auditing firms with Zcash protocol expertise.*

- [librustzcash Security Advisories](https://github.com/zcash/librustzcash/security/advisories) - Official CVE disclosures and patches for core Zcash Rust libraries.
- [Zebra Security Policy](https://github.com/ZcashFoundation/zebra/security/policy) - Vulnerability disclosure guidelines for the Zcash Foundation's Rust node.
- [Halo 2 Security Policy](https://github.com/zcash/halo2/security) - Security reporting and circuit soundness advisories for `halo2`.
- **Auditors with Prior Zcash Protocol Experience:**
  - **Least Authority** — Audited Sapling, Orchard, and Halo 2 specifications.
  - **NCC Group** — Conducted core protocol and consensus cryptography reviews.
  - **Trail of Bits** — Audited Zcash Rust implementations and wallet cryptography.

---

## Video Talks, Workshops & Conferences

*Technical talks from Zcon conferences and core engineering teams.*

- [Zcash Foundation YouTube Channel](https://www.youtube.com/channel/UCi01v05DNTUEC_eB0c9rpgQ/videos) - Full archive of **Zcon** keynotes, cryptography workshops, Zebra engineering talks, and ecosystem panels.
- [ZecHub YouTube & Visual Guides](https://github.com/ZecHub/zechub) - Community-produced walkthroughs on wallets, privacy best practices, and Zcash tooling.

---

## Grants, Funding & Bounties

*Non-dilutive funding opportunities for engineers, researchers, and product builders.*

- [Zcash Community Grants (ZCG)](https://zcashcommunitygrants.org/) - Protocol-funded grant committee (8% of block rewards via ZIP-1014/1015/1016) funding wallets, SDKs, infrastructure, and Zapps.
- [ZCG Open RFPs & Wishlist](https://zcashcommunitygrants.org/) - Active Requests for Proposals including FROST wallets, Zcash Shielded Assets (ZSAs), hardware wallet support, and light client privacy.
- [Z.Cash Ecosystem Grants Portal](https://z.cash/ecosystem/zcash-community-grants/) - Overview of open and invite-only funding pathways across the Zcash ecosystem.

---

## Communities & Working Groups

- [Zcash Community Forum](https://forum.zcashcommunity.com/) - Primary venue for technical RFCs, grant proposals, ZCG discussions, and protocol upgrades.
- [Zcash Foundation GitHub](https://github.com/zcashfoundation) - Open-source repositories, engineering issues, and FROST/Zebra working groups.
- [Shielded Labs GitHub](https://github.com/ShieldedLabs) - Independent Swiss organization funding Crosslink (hybrid PoS) and sustainability upgrades.
- [ZingoLabs GitHub](https://github.com/zingolabs) - Open-source engineering team building `Zaino`, `zingolib`, and next-generation Zcash wallets.

---

## Contributing

Contributions are welcome! To keep this index high-signal for engineers:
1. **Verify maintenance status:** Only submit repositories with active commits or tagged releases compatible with current network upgrades.
2. **Follow the format:** `- [Project Name](https://url) - Concise technical description of what it does and who it is for.`
3. **Include security/deprecation notes:** If a tool only supports transparent addresses (`t-addrs`) or legacy `zcashd` RPCs, state that explicitly.
