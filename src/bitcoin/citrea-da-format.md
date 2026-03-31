# Citrea DA Transaction Format

Citrea serializes a Borsh enum called `DataOnDa` inside taproot script-path reveals on Bitcoin.

## The Five Variants

| Variant | ID | Payload | Purpose |
|---|---|---|---|
| **Complete** | 0 | `Vec<u8>` (compressed proof) | Full ZK batch proof in one tx |
| **Aggregate** | 1 | `Vec<[u8;32]>` txids + wtxids | References chunk txs for large proofs |
| **Chunk** | 2 | `Vec<u8>` (fragment) | Fragment of a large proof |
| **BatchProofMethodId** | 3 | Method ID + signatures + pubkeys | Security council metadata |
| **SequencerCommitment** | 4 | Merkle root + index + L2 end block | Most common — anchors L2 batch to Bitcoin |

## Tapscript Layout

```text
PUSH32 <x_only_pubkey>           32 bytes — Citrea DA pubkey
OP_CHECKSIGVERIFY
PUSH2  <kind_bytes_le>           2 bytes LE — transaction kind (u16)
OP_FALSE
OP_IF
  PUSH <schnorr_signature>       64 bytes
  PUSH <signer_pubkey>           33 bytes compressed
  PUSH <body_chunk> ...          one or many pushdata entries
OP_ENDIF
PUSH8  <nonce_le>                8 bytes LE — wtxid mining nonce
OP_NIP
```

## Identification

Citrea reveal transactions are identified by their **wtxid prefix** — the sequencer picks a nonce so the wtxid starts with `0x0202` (production). This allows fast filtering without full parsing.

## Implementation Notes

- Borsh discriminant is **1 byte** (not 4)
- Body may be split into ≤520-byte pushdata chunks — concatenate before deserializing
- Always compute **wtxid** (witness hash), not txid
- Support `OP_PUSHBYTES_N`, `OP_PUSHDATA1`, and `OP_PUSHDATA2` encodings
