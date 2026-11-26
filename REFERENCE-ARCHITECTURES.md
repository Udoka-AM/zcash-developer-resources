# Production Reference Architectures & Shielded App Patterns

Zcash does not execute general-purpose smart contracts on L1. Instead, production Zcash applications compose **shielded notes, encrypted 512-byte memos, diversified Unified Addresses, and viewing keys** under existing consensus rules.

This guide reverse-engineers two canonical references:
1. **[Zodl](https://github.com/zodl-inc)** (formerly Zashi, maintained by Zcash Open Development Lab) — a production mobile wallet architecture.
2. **[Applied Zcash](https://github.com/appliedzcash/appliedzcash.com)** (`appliedzcash.com`, authored by Mark Henderson & contributors) — a catalog of consensus-native application patterns for Zcash builders.

---

## Part 1: Reverse-Engineering Zodl's Production Wallet Architecture

Zodl ([`zodl-inc/zodl-ios`](https://github.com/zodl-inc) & [`zodl-inc/zodl-android`](https://github.com/zodl-inc/zodl-android)) demonstrates how a modern Zcash client bridges native mobile UIs with Rust cryptography and network-level privacy:

```text
┌──────────────────────────────────────────────────────────────────────┐
│                    Zodl Mobile Client (Swift / Kotlin)               │
│  • UI State, Biometric Keystore, ZIP-321 Payment Request Parsing     │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │ FFI Bridge (UniFFI / C ABI)
┌──────────────────────────────────▼───────────────────────────────────┐
│               Core Rust Engine (librustzcash + SQLite)               │
│  • zcash_client_backend: Orchard/Sapling note commitment tree        │
│  • zcash_client_sqlite: Local encrypted note & nullifier cache       │
│  • Local trial decryption using Unified Full Viewing Key (UFVK)      │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │ Encapsulated gRPC over OHTTP
┌──────────────────────────────────▼───────────────────────────────────┐
│              Oblivious HTTP (OHTTP) Relay ──▶ Gateway                │
│  • Strips client IP address before reaching ZODL wallet servers      │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │
┌──────────────────────────────────▼───────────────────────────────────┐
│                 Wallet Backend (lightwalletd / Zaino)                │
│  • Serves compact blocks & broadcasts signed v5 transactions         │
└──────────────────────────────────────────────────────────────────────┘
```

### Key Production Design Decisions in Zodl
1. **Orchard-First Shielded Default:** Zodl prioritizes Orchard shielded pools and Unified Addresses (`u1...`) by default rather than treating shielding as an optional secondary mode.
2. **Local SQLite Note Commitment Tree (`zcash_client_sqlite`):** Mobile devices cannot hold full blockchain state. Zodl persists compact block headers, witness merkle paths, and spent nullifiers in an encrypted local SQLite database via `librustzcash` FFI bindings, allowing instant spend proof generation without querying a server for balance state.
3. **Network-Layer Privacy via Oblivious HTTP (OHTTP):** In v3.20.0+, ZODL introduced OHTTP-backed wallet servers. Client gRPC requests are encrypted to the gateway key and routed through an oblivious relay—ensuring the relay sees the user's IP (but not the request) while the Zcash wallet server sees the request (but not the user's IP).
4. **Threat-Model-Driven Key Isolation:** As documented in [`zodl-project/wallet_threat_model.md`](https://github.com/Electric-Coin-Company/zashi/blob/main/wallet_threat_model.md), spending keys remain locked inside hardware-backed mobile enclaves while viewing keys drive background compact-block trial decryption.

---

## Part 2: Applied Zcash Patterns (Building Apps Without Smart Contracts)

Mark Henderson's **[Applied Zcash](https://appliedzcash.com/)** ([`appliedzcash/appliedzcash.com`](https://github.com/appliedzcash/appliedzcash.com)) codifies the core primitives every Zcash backend engineer should master:

### Pattern 1: Address-Per-Customer via Diversified Unified Addresses
* **Reference:** [Applied Zcash — Address Per Customer](https://appliedzcash.com/basics/address-per-customer/)
* **How It Works:** Under ZIP-32 and ZIP-316, a single **Unified Full Viewing Key (UFVK)** can derive billions of unlinkable diversified Unified Addresses (`u1...`) using incrementing diversifier indices.
* **Production Use Case:** E-commerce checkouts, SaaS billing, and exchange deposits. Your backend generates a fresh diversified address per invoice/user, scans incoming blocks with a single UFVK, and maps the matched receiver directly to the customer order—with zero on-chain address reuse.

### Pattern 2: Memo-Based Authentication ("Log In with Zcash")
* **Reference:** [Applied Zcash — Log in with Zcash](https://appliedzcash.com/patterns/log-in-with-zcash/)
* **How It Works:** Because every shielded Orchard note carries an encrypted 512-byte memo field, a user can authenticate or bind an off-chain session by sending a minimal shielded payment (or responding to a ZIP-321 payment URI) containing a server-issued challenge nonce in the memo—or receiving a one-time login token back in a shielded memo.
* **Production Use Case:** Pseudonymous account recovery, paid API key provisioning, and spam-resistant messaging without exposing user wallet balances.

### Pattern 3: "Polite Actions" & Scan-Cost Budgeting
* **Reference:** [Applied Zcash — Polite Actions](https://appliedzcash.com/patterns/polite-actions/)
* **How It Works:** Every Zcash light wallet must trial-decrypt every Orchard action in every block to discover incoming funds. High-frequency applications that bloat blocks with unnecessary dummy outputs impose a global CPU/battery tax on every mobile wallet on the network.
* **Production Use Case:** Batching payouts into multi-output v5 transactions rather than broadcasting hundreds of single-action transactions, keeping global trial-decryption overhead low.

### Pattern 4: Off-Chain Shielded Key Signatures (Wishlist / Emerging)
* **Reference:** [Applied Zcash — Signature Login](https://appliedzcash.com/wishlist/signature-login/)
* **How It Works:** Deriving deterministic, domain-separated authentication signatures from shielded key material (via viewing key derivation or dedicated app-key derivation) so users can prove key ownership off-chain without paying a transaction fee or waiting for block confirmation.
