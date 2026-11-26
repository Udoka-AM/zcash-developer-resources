# Local Zcash Devnet: Zebra + Zaino Sandbox

Running a local Zcash stack eliminates multi-hour mainnet sync times and lets you test shielded Orchard transactions, Unified Addresses (`uregtest1...`), and gRPC light-client flows deterministically.

---

## Architecture Overview

```text
┌──────────────────────┐      JSON-RPC (18232)      ┌──────────────────────┐      gRPC (9067)      ┌──────────────────────┐
│        Zebra         │ ─────────────────────────▶ │        Zaino         │ ────────────────────▶ │  Wallet / App SDK    │
│  (Rust Full Node)    │   Block & Mempool State    │   (Rust Indexer)     │   CompactTxStreamer   │ (WebZjs / librust)   │
└──────────────────────┘                            └──────────────────────┘                       └──────────────────────┘
```

1. **Zebra (`ZcashFoundation/zebra`):** Validates consensus rules and exposes JSON-RPC on port `18232`.
2. **Zaino (`zingolabs/zaino`):** Indexes blocks from Zebra and serves `CompactTxStreamer` gRPC endpoints on port `9067` (replacing legacy Go `lightwalletd`).

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
      - ./zebra.toml:/etc/zebrad/zebrad.toml:ro
    command: ["zebrad", "-c", "/etc/zebrad/zebrad.toml", "start"]
    healthcheck:
      test: ["CMD", "curl", "-sf", "[http://127.0.0.1:18232](http://127.0.0.1:18232)"]
      interval: 5s
      timeout: 3s
      retries: 10

  zaino:
    image: zingolabs/zaino:latest
    container_name: zcash-zaino-indexer
    depends_on:
      zebra:
        condition: service_healthy
    ports:
      - "9067:9067"
    environment:
      - ZAINO_NETWORK=Regtest
      - ZAINO_VALIDATOR_RPC_ADDRESS=zebra:18232
      - ZAINO_GRPC_LISTEN_ADDRESS=0.0.0.0:9067
    volumes:
      - zaino-data:/var/lib/zaino

volumes:
  zebra-data:
  zaino-data:
```

### `zebra.toml` (Regtest + NU6/Orchard Enabled)

```toml
[network]
network = "Testnet"

[network.testnet_parameters]
network_name = "Regtest"
network_magic = [170, 232, 63, 95]
slow_start_interval = 0

[network.testnet_parameters.activation_heights]
BeforeOverwinter = 1
Overwinter = 1
Sapling = 1
Blossom = 1
Heartwood = 1
Canopy = 1
NU5 = 1
NU6 = 1

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
  [http://127.0.0.1:18232](http://127.0.0.1:18232)
```

Verify Zaino's gRPC endpoint is serving compact blocks (requires `grpcurl`):
```bash
grpcurl -plaintext 127.0.0.1:9067 cash.z.wallet.sdk.rpc.CompactTxStreamer/GetLatestBlock
```

---

## 3. Connecting Browser Apps (`WebZjs`) Locally

Browsers cannot speak raw HTTP/2 gRPC directly to port `9067`. Place an **Envoy** or **gRPC-Web** proxy in front of Zaino on port `8080` when developing with `@chainsafe/webzjs-wallet`, and pass `"http://localhost:8080"` as your `GRPC_WEB_PROXY_URL`.
