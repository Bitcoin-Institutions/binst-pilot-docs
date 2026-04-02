# Taproot Reader (Rust)

A Rust workspace that decodes BINST data directly from Bitcoin. Four crates, 54 tests.

## Crate Architecture

```text
taproot-reader/
  crates/
    citrea-decoder/    ← Citrea DA inscription parser (no_std, WASM-ready)
    binst-decoder/     ← Storage slots → protocol entities (JMT key parser, forward-hash lookup)
    binst-inscription/ ← Ordinals envelope parser for binst metaprotocol
    cli/               ← citrea-scanner binary (Bitcoin Core RPC + Citrea RPC)
```

## citrea-decoder

Parses Citrea DA inscriptions from raw tapscript witness data. Handles all five `DataOnDa` variants, pushdata chunking, and wtxid prefix filtering. Also decodes batch proof output (Brotli decompression, journal extraction, state diff parsing).

- `no_std` compatible, WASM-ready
- 7 tests

## binst-decoder

Maps L2 storage slot diffs to BINST entities. Given a state diff from a batch proof, reconstructs `InstitutionState`, `ProcessTemplateState`, etc. Parses Citrea JMT keys and builds forward-hash lookup tables for matching state diff entries to known BINST contracts.

- Computes Solidity storage slot positions (keccak256-based)
- JMT key parsing: `E/s/` (storage), `E/H/` (headers), `E/a/` (accounts)
- Forward-hash lookup: SHA-256 precomputation of all known (address, slot) pairs
- Carries `BitcoinIdentity` struct linking entities across layers
- 27 tests + 5 e2e integration tests

## binst-inscription

Parses Ordinals envelopes for `binst` metaprotocol inscriptions. Extracts typed entity bodies (institution, template, instance, step execution).

- Validates metaprotocol field, content type, parent chain
- 10 tests

## cli (citrea-scanner)

Binary that scans for Citrea DA transactions. Supports two modes:

- **Bitcoin Core mode**: connects to a local full node via RPC
- **Citrea RPC mode**: queries batch proofs directly from a Citrea node (no Bitcoin node required)

The `--discover` flag auto-discovers all BINST contracts by crawling the deployer on-chain.

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

See also: [BitcoinIdentity Type](./bitcoin-identity.md)
