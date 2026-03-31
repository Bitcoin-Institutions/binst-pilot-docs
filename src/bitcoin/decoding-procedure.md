# Decoding Procedure

Step-by-step process for finding and decoding Citrea DA inscriptions from Bitcoin.

## 1. Choose a Scanning Strategy

- **Tip-based** (continuous): scan from last seen block to current tip
- **Range scan** (investigation): specific block range with `--from`/`--to`
- **Targeted**: only blocks with `nTx > 1` (many testnet blocks are coinbase-only)

## 2. Fetch the Block

Use `getblockhash(height)` then `getblock(hash)` to get the full block with witness data.

## 3. Filter by wtxid Prefix

For each non-coinbase transaction, compute `wtxid = SHA256d(serialized_tx_with_witness)`. Check if it starts with `0x0202`.

## 4. Extract Tapscript

For script-path spends, the tapscript is the second-to-last witness element: `witness[witness.len() - 2]`.

## 5. Parse Structured Tapscript

Walk pushdata opcodes in order: pubkey → signature → signer → body chunks. Concatenate body chunks.

## 6. Borsh Deserialize

First byte = variant discriminant (0–4). Deserialize to typed `DataOnDa` value.

## 7. Map to Protocol Events

| Variant | What it means |
|---|---|
| SequencerCommitment | New L2 batch finalized up to `l2_end_block_number` |
| Complete | Full ZK batch proof — verify to confirm correctness |
| Aggregate + Chunk | Large proof assembly instructions |
| BatchProofMethodId | Security council / method-ID update |

## CLI Usage

```bash
# Scan a single block
cargo run --bin citrea-scanner -- --block 127600 --rpc-user testnet4rpc --rpc-pass <pass>

# Scan a range, JSON output
cargo run --bin citrea-scanner -- --from 127600 --to 127761 --format json
```

See the `taproot-reader/crates/cli/` directory in the [pilot repository](https://github.com/Bitcoin-Institutions/binst-pilot).
