# Local Zcash Devnet: Zebra + Zaino Sandbox

Running a local Zcash stack eliminates multi-hour mainnet sync times and lets you test shielded Orchard transactions, Unified Addresses (`uregtest1...`), and gRPC light-client flows deterministically.

---

## Architecture Overview

```text
┌──────────────────────┐      JSON-RPC (18232)      ┌──────────────────────┐      gRPC (8137)      ┌──────────────────────┐
│        Zebra         │ ─────────────────────────▶ │        Zaino         │ ────────────────────▶ │  Wallet / App SDK    │
│  (Rust Full Node)    │   Block & Mempool State    │   (Rust Indexer)     │   CompactTxStreamer   │ (WebZjs / librust)   │
└──────────────────────┘                            └──────────────────────┘                       └──────────────────────┘
```

1. **Zebra (`ZcashFoundation/zebra`):** Validates consensus rules and exposes JSON-RPC on port `18232`.
2. **Zaino (`zingolabs/zaino`):** Indexes blocks from Zebra and serves `CompactTxStreamer` gRPC endpoints on port `8137` (replacing legacy Go `lightwalletd`).

---

## 1. Docker Compose Setup (`docker-compose.yml`)

Save this in a local `devnet/` folder alongside a `zebra.toml` config file:

```yaml
services:
  zebra:
    image: zfnd/zebra:latest
    container_name: zcash-zebra-regtest
    ports:
      - "18232:18232"
    volumes:
      - zebra-data:/home/zebra/.cache/zebra
      - ./zebra.toml:/home/zebra/.config/zebrad.toml:ro
    healthcheck:
      test: ["CMD", "curl", "-sf", "-H", "content-type: application/json", "--data-binary", "{\"jsonrpc\":\"2.0\",\"id\":\"hc\",\"method\":\"getinfo\",\"params\":[]}", "http://127.0.0.1:18232"]
      interval: 5s
      timeout: 3s
      retries: 10

  zaino:
    # The no-TLS build is for local development only. Use the TLS image (zingodevops/zaino) in production.
    image: zingodevops/zaino:0.10.2-no-tls
    container_name: zcash-zaino-indexer
    depends_on:
      zebra:
        condition: service_healthy
    ports:
      - "8137:8137"
    environment:
      - ZAINO_NETWORK=Regtest
      - ZAINO_VALIDATOR_SETTINGS__VALIDATOR_JSONRPC_LISTEN_ADDRESS=zebra:18232
      - ZAINO_GRPC_SETTINGS__LISTEN_ADDRESS=0.0.0.0:8137
    volumes:
      - zaino-data:/app/data

volumes:
  zebra-data:
  zaino-data:
```

### `zebra.toml` (Regtest)

Zebra activates every network upgrade at height 1 on Regtest by default. Add a `[network.testnet_parameters.activation_heights]` section only to test a different schedule. The `[mining]` address is required for the `generate` RPC.

```toml
[network]
network = "Regtest"

[mining]
miner_address = "t27eWDgjFYJGVXmzrXeVjnb5J3uXDM9xH9v"

[rpc]
listen_addr = "0.0.0.0:18232"
enable_cookie_auth = false

[state]
cache_dir = "/home/zebra/.cache/zebra"
```

---

## 2. Booting & Mining Regtest Blocks

Start the stack:
```bash
docker compose up -d
```

Mine 101 blocks (required for coinbase maturity) directly via Zebra's JSON-RPC:
```bash
curl -s --data-binary '{"jsonrpc":"1.0","id":"mine","method":"generate","params":[101]}' \
  -H 'content-type: application/json' \
  http://127.0.0.1:18232
```

Verify Zaino's gRPC endpoint is serving compact blocks (requires `grpcurl`):
```bash
grpcurl -plaintext 127.0.0.1:8137 cash.z.wallet.sdk.rpc.CompactTxStreamer/GetLatestBlock
```

---

## 3. Connecting Browser Apps (`WebZjs`) Locally

Browsers cannot speak raw HTTP/2 gRPC directly to port `8137`. Place an **Envoy** or **gRPC-Web** proxy in front of Zaino on port `8080` when developing with `@chainsafe/webzjs-wallet`, and pass `"http://localhost:8080"` as your `GRPC_WEB_PROXY_URL`.
