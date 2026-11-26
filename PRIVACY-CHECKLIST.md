# Zcash Privacy Engineering & OpSec Checklist

Zcash's Orchard zk-SNARKs hide sender, receiver, and amount on-chain—but **application-layer and network-layer leaks** can still deanonymize users. Audit every production Zcash app against these five failure modes.

---

## 1. Network-Layer Metadata Leaks (IP Correlation)

* **The Leak:** A light wallet queries `GetTransaction` or broadcasts a raw transaction (`SendTransaction`) directly to a single `lightwalletd`/`Zaino` server from the user's home IP address. The server operator can correlate the user's IP with specific txids and timing.
* **Production Fix:**
  * Route light-client gRPC requests over **Oblivious HTTP (OHTTP)** relays or **Tor** so the wallet server sees the request payload without learning the client's IP address.
  * Sync strictly via **compact block ranges** (`GetBlockRange`) and perform trial decryption locally on-device—never ask the server whether a specific address or note belongs to the user.

---

## 2. Cross-Pool Linking (Transparent ↔ Shielded)

* **The Leak:** A user receives funds to a transparent receiver (`t1...`) inside a Unified Address and immediately auto-shields the exact balance in the very next block. An on-chain observer can link the transparent deposit to the shielded transaction via exact amount matching (minus standard fee) and block proximity.
* **Production Fix:**
  * Default to **Orchard-only receivers** (`u1...`) whenever the counterparty supports Unified Addresses.
  * When shielding transparent UTXOs is required, surface explicit UI warnings about amount/timing correlation or introduce randomized shielding delays.

---

## 3. Memo Field Length & Formatting Leaks (ZIP-302)

* **The Leak:** Every Zcash shielded output includes a **512-byte encrypted memo field**. While the contents are encrypted on-chain, applications that write variable-length plaintext or unpadded JSON off-chain (or leak memo structure via error responses) expose metadata.
* **Production Fix:**
  * Always pad structured application memos to the full 512 bytes using standard ZIP-302 zero-padding (`0x00` bytes after UTF-8 or binary payload).
  * Validate memo schemas strictly inside your application backend without returning verbose parse errors to unauthenticated callers.

---

## 4. Wallet Birthday Height Omission

* **The Leak / UX Failure:** Initializing a new wallet or watch-only service without recording the creation block height (`birthday_height`) forces `librustzcash` / `WebZjs` to trial-decrypt millions of historical blocks from Sapling/Orchard activation—stalling sync for minutes or hours and hammering RPC nodes.
* **Production Fix:**
  * Persist `birthday_height` alongside every generated seed phrase or Unified Full Viewing Key (UFVK).
  * Enforce birthday-height inputs during wallet restore flows.

---

## 5. Viewing Key Over-Sharing (`UFVK` vs. `UIVK`)

* **The Leak:** Giving a third-party analytics tool, merchant checkout server, or auditor a **Unified Full Viewing Key (UFVK)** exposes both **incoming and outgoing** transaction history for that account.
* **Production Fix:**
  * Follow the principle of least privilege: export a **Unified Incoming Viewing Key (UIVK)** when a service only needs to detect incoming deposits, reserving UFVKs strictly for trusted watch-only accounting.
