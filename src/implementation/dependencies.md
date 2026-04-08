# Dependencies

## `rust-bitcoin` (v0.32)

The WASM webapp uses [`rust-bitcoin`](https://github.com/rust-bitcoin/rust-bitcoin)
v0.32 as its Bitcoin primitive library. This is the only crate in the
project that works directly with Bitcoin transaction types in Rust.

```toml
bitcoin = { version = "0.32", features = ["serde", "base64"] }
```

### What it provides

| Type / function | Used for |
|----------------|----------|
| `bitcoin::Transaction` | Constructing commit and reveal transactions |
| `bitcoin::psbt::Psbt` | Building partially-signed transactions for wallet signing |
| `bitcoin::taproot::TaprootSpendInfo` | Computing the Taproot output key from the inscription script leaf |
| `bitcoin::key::XOnlyPublicKey` | Validating the admin public key before inscription |
| `bitcoin::Address` | Deriving commit output and change addresses from the connected pubkey |
| `bitcoin::ScriptBuf` | Building Tapscript inscription envelopes (OP_FALSE OP_IF … OP_ENDIF) |
| `bitcoin::Amount` / `bitcoin::OutPoint` | UTXO representation for fee calculation |
| `bitcoin::Network` | Switching between Testnet4, Signet, and Mainnet address formats |

### Why v0.32 specifically

v0.32 introduced `bitcoin::psbt::Psbt` with the updated BIP 174
API. The webapp's `txbuilder.rs` uses `Psbt::serialize_hex()` and
`Psbt::from_str()` for the round-trip test. v0.31 and earlier have
a different PSBT serialization API.

### Feature flags

- `serde` — enables `Serialize`/`Deserialize` on Bitcoin types, used
  when passing inscription data across the WASM boundary as JSON
- `base64` — enables PSBT base64 encoding, which is the standard
  format wallets (UniSat, SafePal) expect when receiving a PSBT

---

## WASM webapp dependency graph

```text
binst-pilot-webapp
├── bitcoin 0.32 (serde, base64)          ← Bitcoin transaction primitives
├── binst-inscription  ─── git/[patch]    ← Parse binst metaprotocol inscriptions
├── binst-decoder      ─── git/[patch]    ← Protocol state reconstruction
│   └── citrea-decoder ─── path dep
├── wasm-bindgen 0.2                      ← WASM ↔ JS boundary
├── wasm-bindgen-futures 0.4              ← async/await in WASM
├── web-sys 0.3                           ← DOM, fetch, console, storage APIs
├── js-sys 0.3                            ← JsValue, Promise
├── serde + serde_json                    ← JSON serialization
└── html-escape 0.2                       ← XSS-safe HTML generation
```

### Protocol crate feature flags

`binst-decoder` ships with two feature sets:

| Feature | Purpose |
|---------|---------|
| `std` (default) | Standard library — for CLI and native tests |
| `wasm` | Enables `wasm-bindgen` exports and `serde_json` for WASM use |

The webapp uses `default-features = false, features = ["wasm"]` to
avoid pulling in `std`-only code into the WASM binary.

---

## `rust-bitcoin` vs the protocol crates

The split is intentional:

| Responsibility | Crate |
|----------------|-------|
| **Parse** inscription JSON from Bitcoin witness | `binst-inscription` (no `rust-bitcoin` dep) |
| **Decode** Citrea DA state diffs | `citrea-decoder` / `binst-decoder` (no `rust-bitcoin` dep) |
| **Build** commit+reveal PSBTs for wallet signing | `binst-pilot-webapp` via `txbuilder.rs` (uses `rust-bitcoin`) |

The protocol crates stay `no_std`-compatible and do not depend on
`rust-bitcoin`. Only the webapp's transaction builder needs Bitcoin
primitives, and it is the only place they are used.
