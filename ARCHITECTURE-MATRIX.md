# Zcash Modern (Rust) vs. Legacy (C++) Architecture Matrix

Zcash is completing a multi-year transition away from its original C++ monolithic node (`zcashd`) toward a modular, memory-safe Rust stack.

| Layer | Modern Production Stack | Legacy Stack (Deprecated / Maintenance) | Why Migrate? |
| :--- | :--- | :--- | :--- |
| **Consensus Node** | **Zebra** (`ZcashFoundation/zebra`) | `zcashd` (`zcash/zcash`) | Independent Rust implementation with cleaner modularity, faster sync, and active consensus R&D. |
| **Wallet Backend** | **Zallet** / **librustzcash** | `zcashd` built-in wallet | Decouples key management and note scanning from consensus validation. |
| **Light Client Indexer** | **Zaino** (`zingolabs/zaino`) | `lightwalletd` (Go) | High-throughput Rust indexer implementing the `CompactTxStreamer` gRPC protocol without Go/C++ glue. |
| **Browser / Web SDK** | **WebZjs** (`ChainSafe/WebZjs`) | Custom RPC proxies | Multi-threaded Wasm trial decryption directly in browser WebWorkers. |
| **Multi-Sig / Custody** | **FROST** (`ZcashFoundation/frost`) | Transparent P2SH multisig | Threshold Schnorr signatures supporting shielded Orchard spending. |
