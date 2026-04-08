# ZK Batch Proofs — Computational Integrity

The L2 (Citrea) periodically writes **ZK batch proofs** to Bitcoin — mathematical guarantees that every state transition was computed correctly.

## How It Works

1. Every BINST transaction lives in a Citrea L2 block
2. The sequencer inscribes **Sequencer Commitments** (Merkle roots) on Bitcoin — pins ordering
3. The batch prover inscribes **ZK proofs** (Groth16 via RISC Zero) on Bitcoin with state diffs — proves correctness
4. Anyone with a Bitcoin node can **reconstruct the entire L2 state** including all BINST data

## Finality Levels

| Level | What happens | How to verify |
|---|---|---|
| **Soft Confirmation** | Sequencer signs the L2 block | Transaction receipt |
| **Committed** | Sequencer commitment inscribed on Bitcoin | `citrea_getLastCommittedL2Height` |
| **ZK-Proven** | ZK batch proof inscribed on Bitcoin | `citrea_getLastProvenL2Height` |

## Why This Matters for BINST

Process instance state reaches Bitcoin via batch proof. This means:

- A second L2 can verify process execution by **reading Bitcoin DA** — no messaging protocol needed
- The `binst-protocol` CLI can reconstruct which steps were executed, by whom, at what timestamp
- Cross-chain execution verification is **trustless** — it goes through Bitcoin, not through any relay

See: [Citrea DA Format](./citrea-da-format.md) · [Decoding Procedure](./decoding-procedure.md)
