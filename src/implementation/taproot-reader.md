# BINST Protocol (Rust)

The Rust protocol crates live in a **standalone repository**:
[github.com/Bitcoin-Institutions/BINST-protocol](https://github.com/Bitcoin-Institutions/BINST-protocol)

This separation means any project — CLI tooling, WASM webapp, future
mobile app — can depend on the protocol crates via a single git
dependency, with a local `[patch]` override for development.

## Repository structure

```text
binst-protocol/          ← standalone Cargo workspace
  Cargo.toml             ← workspace root (resolver = "2")
  crates/
    citrea-decoder/      ← Citrea DA inscription parser (no_std, WASM-ready)
    binst-decoder/       ← Storage slots → protocol entities + miniscript vault
    binst-inscription/   ← Ordinals envelope parser for binst metaprotocol
    cli/                 ← citrea-scanner binary (Bitcoin Core RPC + Citrea RPC)
  schema/                ← JSON schema for binst metaprotocol entities
  BITCOIN-IDENTITY.md
  DECODING.md
  conceptual.md
```

Previously these crates lived inside `binst-pilot/taproot-reader/`.
They were extracted into their own repository to be reusable as a
dependency of the WASM webapp and any future project.

## Using the crates

### In a project (git dependency)

```toml
[dependencies]
binst-inscription = { git = "https://github.com/Bitcoin-Institutions/BINST-protocol.git" }
binst-decoder     = { git = "https://github.com/Bitcoin-Institutions/BINST-protocol.git",
                      default-features = false, features = ["wasm"] }
```

### Local development override ([patch])

```toml
[patch."https://github.com/Bitcoin-Institutions/BINST-protocol.git"]
binst-inscription = { path = "../../binst-protocol/crates/binst-inscription" }
binst-decoder     = { path = "../../binst-protocol/crates/binst-decoder" }
citrea-decoder    = { path = "../../binst-protocol/crates/citrea-decoder" }
```

The `[patch]` block redirects git deps to the local checkout so you
can iterate on protocol crates and the webapp simultaneously without
pushing. Remove or comment it out when testing against the published repo.

## `citrea-decoder`

Parses Citrea DA inscriptions from raw tapscript witness data. Handles
all five `DataOnDa` variants, pushdata chunking, and wtxid prefix
filtering. Also decodes batch proof output (Brotli decompression,
journal extraction, state diff parsing).

- `no_std` compatible, WASM-ready
- 7 tests

## `binst-decoder`

Maps L2 storage slot diffs to BINST entities. Given a state diff from
a batch proof, reconstructs `InstitutionState`, `ProcessTemplateState`,
etc. Parses Citrea JMT keys and builds forward-hash lookup tables for
matching state diff entries to known BINST contracts.

- Computes Solidity storage slot positions (keccak256-based)
- JMT key parsing: `E/s/` (storage), `E/H/` (headers), `E/a/` (accounts)
- Forward-hash lookup: SHA-256 precomputation of all known (address, slot) pairs
- **Human-readable value decoding** (`value` module): decodes raw Citrea
  LE storage values to addresses, uints, bools, Solidity strings, and
  packed `StepState` structs
- Key discovery: Citrea stores EVM slot values in little-endian word
  order (entire 32-byte word byte-reversed vs. standard Solidity ABI)
- Carries `BitcoinIdentity` struct linking entities across layers
- **Miniscript vault module** (`vault` module): compiles BIP 379 spending
  policies to Taproot descriptors using `rust-miniscript`. Generates
  wallet-compatible descriptors, derives addresses, and analyzes spending
  paths. WASM-exportable for in-browser vault generation.
- 52 tests (27 unit + 5 e2e + 9 value decoding + 11 vault)

## `binst-inscription`

Parses Ordinals envelopes for `binst` metaprotocol inscriptions.
Extracts typed entity bodies (institution, template, instance, step
execution).

- Validates metaprotocol field, content type, parent chain
- 10 tests

## `cli` (`citrea-scanner`)

Binary that scans for Citrea DA transactions. Supports two modes:

- **Bitcoin Core mode**: connects to a local full node via RPC
- **Citrea RPC mode**: queries batch proofs directly from a Citrea node
  (no Bitcoin node required)

The `--discover` flag auto-discovers all BINST contracts by crawling
the deployer on-chain.

```bash
# Bitcoin Core mode (uses cookie auth by default)
cargo run --bin citrea-scanner -- --block 127600

# Citrea RPC mode with auto-discovery
cargo run --bin citrea-scanner -- \
  --citrea-rpc https://rpc.testnet.citrea.xyz \
  --discover \
  --deployer 0xd0abca83bd52949fcf741d6da0289c5ec7235aaf \
  --block 127848
```

- 5 tests

```bash
cd binst-protocol && cargo test    # runs all 79 protocol tests
```

See also: [BitcoinIdentity Type](./bitcoin-identity.md)
