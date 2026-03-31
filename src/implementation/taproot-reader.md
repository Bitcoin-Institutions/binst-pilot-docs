# Taproot Reader (Rust)

A Rust workspace that decodes BINST data directly from Bitcoin. Four crates, 16 tests.

## Crate Architecture

```text
taproot-reader/
  crates/
    citrea-decoder/    ← Citrea DA inscription parser (no_std, WASM-ready)
    binst-decoder/     ← Storage slots → protocol entities (BitcoinIdentity-aware)
    binst-inscription/ ← Ordinals envelope parser for binst metaprotocol
    cli/               ← citrea-scanner binary (Bitcoin Core RPC)
```

## citrea-decoder

Parses Citrea DA inscriptions from raw tapscript witness data. Handles all five `DataOnDa` variants, pushdata chunking, and wtxid prefix filtering.

- `no_std` compatible, WASM-ready
- 2 tests

## binst-decoder

Maps L2 storage slot diffs to BINST entities. Given a state diff from a batch proof, reconstructs `InstitutionState`, `ProcessTemplateState`, etc.

- Computes Solidity storage slot positions (keccak256-based)
- Carries `BitcoinIdentity` struct linking entities across layers
- 5 tests

## binst-inscription

Parses Ordinals envelopes for `binst` metaprotocol inscriptions. Extracts typed entity bodies (institution, template, instance, step execution).

- Validates metaprotocol field, content type, parent chain
- 9 tests

## cli (citrea-scanner)

Binary that connects to Bitcoin Core RPC and scans blocks for Citrea DA transactions.

```bash
cargo run --bin citrea-scanner -- --block 127600 --rpc-user user --rpc-pass pass
cargo run --bin citrea-scanner -- --from 127600 --to 127761 --format json
```

See also: [BitcoinIdentity Type](./bitcoin-identity.md)
