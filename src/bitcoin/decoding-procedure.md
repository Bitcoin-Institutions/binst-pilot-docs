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

### Bitcoin Core mode (local full node)

```bash
# Scan a single block (uses cookie auth by default)
cargo run --bin citrea-scanner -- --block 127600

# Scan a range with explicit auth, JSON output
cargo run --bin citrea-scanner -- --from 127600 --to 127761 --format json \
  --rpc-user <user> --rpc-pass <pass>
```

### Citrea RPC mode (no Bitcoin node required)

```bash
# Query batch proofs via Citrea RPC
cargo run --bin citrea-scanner -- \
  --citrea-rpc https://rpc.testnet.citrea.xyz \
  --discover \
  --factory 0x6a1d2adbac8682773ed6700d2118c709c8ce5000 \
  --block 127848

# Manual instance addresses (skip discovery)
cargo run --bin citrea-scanner -- \
  --citrea-rpc https://rpc.testnet.citrea.xyz \
  --factory 0x... --instance 0x... \
  --from 127840 --to 127850
```

The `--discover` flag crawls the factory contract on-chain:
`factory.getInstanceCount()` → `factory.allInstances(i)` → instance addresses.

## 8. Human-Readable Value Decoding

Once BINST storage slot changes are identified, raw hex values are automatically
decoded into human-readable form. Example output from block 127848:

```
BINSTProcess.templateInscriptionId = "abc123…i0"
BINSTProcess.creator = 0x8cf6fe5cd0905b6bfb81643b0dcda64af32fd762
BINSTProcess.stepStates[0] = Completed by 0x8cf6fe5cd0905b6bfb81643b0dcda64af32fd762
BINSTProcess.currentStepIndex = 4
BINSTProcess.completed = true
BINSTProcess.totalSteps = 4
```

### Supported Solidity types

| Solidity type | Decoded form |
|---|---|
| `address` | `0x8cf6fe5c…d762` |
| `uint256` | `4`, `1774750572` |
| `bool` | `true` / `false` |
| `string` (short ≤31 bytes) | `"KYC Verification"` |
| `string` (long >31 bytes) | `<string, 62 bytes>` |
| `StepState` (packed struct) | `Completed by 0x8cf6…d762` |
| `bytes32` | `0x…` hex |

### Citrea little-endian word order

**Key discovery:** Citrea / Sovereign SDK stores EVM storage values in
little-endian word order — the entire 32-byte slot is byte-reversed compared
to standard Solidity ABI encoding.

```
uint256 value 1:
  Standard Solidity (BE): 0x00000000…00000001
  Citrea state trie (LE): 0x01000000…00000000
```

The decoder pads to 32 bytes (state diffs may trim trailing LE zeros) and
reverses before interpreting according to the field's Solidity type.

### Packed struct layout (StepState)

Solidity packs `uint8 status` + `address actor` right-aligned in one slot:

```
BE word (after LE→BE reversal):
  [00 × 11 bytes][actor × 20 bytes][status × 1 byte]
   ↑ padding      ↑ bytes 11..31    ↑ byte 31
```

Status: `0` = Pending, `1` = Completed, `2` = Rejected.

See the `binst-protocol/crates/cli/` directory in the [pilot repository](https://github.com/Bitcoin-Institutions/binst-pilot).
