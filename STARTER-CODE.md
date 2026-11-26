# Zcash Starter Code: WebZjs & `librustzcash`

Drop-in examples for connecting to Zcash and deriving **ZIP-316 Unified Addresses (`u1...`)** that bundle Orchard, Sapling, and transparent receivers into a single address.

---

## 1. WebZjs (TypeScript / Browser Wasm)

[ChainSafe/WebZjs](https://github.com/ChainSafe/WebZjs) compiles `librustzcash` into WebAssembly and executes shielded trial decryption across browser WebWorkers so the UI stays responsive.

### Installation
`@chainsafe/webzjs-keys` is on npm. `@chainsafe/webzjs-wallet` is not published to npm yet, so build it from source (needs Rust, [`just`](https://github.com/casey/just), `wasm-pack`, and clang 17+):

```bash
npm install @chainsafe/webzjs-keys

git clone https://github.com/ChainSafe/WebZjs.git
cd WebZjs && just build   # writes the wasm packages into ./packages
```

### `wallet-quickstart.ts`
```typescript
import initWasm, {
  initThreadPool,
  WebWallet,
} from "@chainsafe/webzjs-wallet";
import { generate_seed_phrase } from "@chainsafe/webzjs-keys";

async function setupZcashBrowserWallet() {
  // 1. Initialize the Wasm module and WebWorker thread pool (call ONCE per app lifecycle)
  await initWasm();
  await initThreadPool(navigator.hardwareConcurrency || 4);

  // 2. Connect to a gRPC-web proxy for lightwalletd / Zaino
  // Note: Browser environments require a gRPC-web proxy (not a raw HTTP/2 lightwalletd URL)
  const GRPC_WEB_PROXY_URL = "https://zcash-mainnet.chainsafe.dev";
  const NETWORK = "main"; // "main" or "test"
  const MIN_CONFIRMATIONS_TRUSTED = 3;
  const MIN_CONFIRMATIONS_UNTRUSTED = 3;

  // Parameters: network, proxy URL, trusted confs, untrusted confs, serialized db (optional)
  const wallet = new WebWallet(
    NETWORK,
    GRPC_WEB_PROXY_URL,
    MIN_CONFIRMATIONS_TRUSTED,
    MIN_CONFIRMATIONS_UNTRUSTED,
    null
  );

  // 3. Generate a 24-word BIP-39 mnemonic (or pass an existing seed phrase)
  const seedPhrase = generate_seed_phrase();
  const accountIndex = 0;

  // Setting a birthday height skips scanning blocks created before this wallet existed.
  // For a brand-new seed, the current chain tip is the birthday.
  const birthdayHeight = Number(await wallet.get_latest_block());

  const accountId = await wallet.create_account(
    "primary-account",
    seedPhrase,
    accountIndex,
    birthdayHeight
  );

  // 4. Derive the current ZIP-316 Unified Address (u1...)
  const unifiedAddress = await wallet.get_current_address(accountId);
  console.log("Unified Address:", unifiedAddress);

  // 5. Trigger background shielded sync via WebWorkers
  await wallet.sync();

  // 6. Inspect wallet summary (shielded Orchard/Sapling + transparent balances)
  const summary = await wallet.get_wallet_summary();
  console.log("Wallet Summary:", summary);
}

setupZcashBrowserWallet().catch(console.error);
```

### Browser Deployment Headers (Required)
Because WebZjs relies on `SharedArrayBuffer` for multi-threaded Wasm trial decryption, your web server or hosting provider **must** serve Cross-Origin Isolation headers on the main document response:

```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

* **Vite (`vite.config.ts`):**
  ```typescript
  import { defineConfig } from "vite";

  export default defineConfig({
    server: {
      headers: {
        "Cross-Origin-Opener-Policy": "same-origin",
        "Cross-Origin-Embedder-Policy": "require-corp",
      },
    },
  });
  ```

* **Vercel (`vercel.json`):**
  ```json
  {
    "headers": [
      {
        "source": "/(.*)",
        "headers": [
          { "key": "Cross-Origin-Opener-Policy", "value": "same-origin" },
          { "key": "Cross-Origin-Embedder-Policy", "value": "require-corp" }
        ]
      }
    ]
  }
  ```

> **Verification:** Run `window.crossOriginIsolated` in Chrome DevTools Console. It must return `true` before calling `initThreadPool()`.

---

## 2. `librustzcash` (Rust Native / Backend)

For CLI tools, mobile FFI bridges, or server-side infrastructure, use `zcash_keys` and `zcash_protocol` directly from [zcash/librustzcash](https://github.com/zcash/librustzcash).

### `Cargo.toml`
```toml
[package]
name = "zcash-ua-starter"
version = "0.1.0"
edition = "2024"

[dependencies]
zcash_keys = { version = "0.17", features = ["orchard", "sapling", "transparent-inputs"] }
zcash_protocol = "0.11"
zip32 = "0.3"
getrandom = { version = "0.3", features = ["std"] }
```

### `src/main.rs`
```rust
use zcash_keys::keys::{UnifiedAddressRequest, UnifiedSpendingKey};
use zcash_protocol::consensus::Network;
use zip32::AccountId;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 1. Select target network (MainNetwork or TestNetwork)
    let network = Network::MainNetwork;

    // 2. Generate 64 bytes of cryptographic entropy (in production, derive from BIP-39 mnemonic)
    let mut seed = [0u8; 64];
    getrandom::fill(&mut seed)?;

    // 3. Derive the ZIP-32 Unified Spending Key (USK) for Account 0
    // Bundles Orchard, Sapling, and Transparent spending keys
    let account_id = AccountId::try_from(0)?;
    let usk = UnifiedSpendingKey::from_seed(&network, &seed, account_id)
        .map_err(|e| format!("{e:?}"))?;

    // 4. Downgrade to a Unified Full Viewing Key (UFVK)
    // Safe to share with watch-only indexers (Zaino / lightwalletd) without exposing spend authority
    let ufvk = usk.to_unified_full_viewing_key();

    // 5. Derive the default ZIP-316 Unified Address containing Orchard + Sapling + Transparent receivers
    let (unified_address, diversifier_index) =
        ufvk.default_address(UnifiedAddressRequest::AllAvailableKeys)?;

    // 6. Encode as a human-readable Bech32m string (starts with `u1` on mainnet, `utest1` on testnet)
    let encoded_ua = unified_address.encode(&network);

    println!("Account Index:     {}", u32::from(account_id));
    println!("Diversifier Index: {:?}", diversifier_index);
    println!("Unified Address:   {}", encoded_ua);

    Ok(())
}
```
