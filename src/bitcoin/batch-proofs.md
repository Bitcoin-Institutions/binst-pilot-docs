# ZK Batch Proofs — Computational Integrity

The L2 (Citrea) periodically writes **ZK batch proofs** to Bitcoin — mathematical guarantees that every state transition was computed correctly.

## How It Works

1. Every BINST transaction lives in a Citrea L2 block
2. The sequencer inscribes **Sequencer Commitments** (Merkle roots) on Bitcoin — pins ordering
3. The batch prover inscribes **ZK proofs** (Groth16 via RISC Zero) on Bitcoin with state diffs — proves correctness
4. Anyone with a Bitcoin node can **reconstruct the entire L2 state** including all BINST data

## Finality Levels

| Level | What happens | How to verify | Trust assumption |
|---|---|---|---|
| **Soft Confirmation** | Sequencer signs the L2 block | Transaction receipt | Trust sequencer |
| **Committed** | Sequencer commitment inscribed on Bitcoin | `citrea_getLastCommittedL2Height` | Bitcoin consensus + sequencer honesty |
| **ZK-Proven** | ZK batch proof inscribed on Bitcoin | `citrea_getLastProvenL2Height` | Bitcoin consensus + math only |

Each BINST process step execution progresses through these three levels
automatically — no user action required. The webapp tracks and displays
the finality stage of each transaction (instance creation + each step)
in the Execute view's finality card.

### Querying finality programmatically

```text
# Get the last committed and proven L2 heights
citrea_getLastCommittedL2Height  →  { height, idx }
citrea_getLastProvenL2Height     →  { height, idx }

# Get the block number for a specific tx
eth_getTransactionReceipt(txHash)  →  receipt.blockNumber

# Classification:
if tx_block <= proven_height    →  "Proven"     (green)
if tx_block <= committed_height →  "Committed"  (blue)
otherwise                       →  "Soft Confirmation" (yellow)
```

### Event log recovery

When the webapp loads an instance that was created before finality
tracking was added, it recovers tx hashes by scanning on-chain event
logs from the factory (`InstanceCreated`) and instance (`StepExecuted`)
contracts, using `eth_getLogs` in 1000-block chunks scanning backwards
from the current block.

## BitVM2 and the Clementine Bridge

Citrea's **Clementine Bridge** implements **BitVM2** — a trust-minimized
fraud-proof system that secures the BTC ↔ cBTC peg:

```text
Operator posts assertion about L2 state on Bitcoin
  ├─ No challenge within timeout → assertion accepted (optimistic case)
  └─ Challenger disputes → operator must reveal intermediate steps
       ├─ Operator correct → challenger loses bond
       └─ Operator wrong  → operator's bond slashed on Bitcoin L1
```

**Trust model:** 1-of-N honesty — only **one** honest verifier is needed
to catch fraud. This is stronger than multisig bridges (which require
M-of-N honesty) and weaker only than a native Bitcoin covenant (which
requires zero trust beyond Bitcoin consensus).

### Why BitVM2 matters for BINST

Every `BINSTProcess.executeStep()` call on Citrea is:

1. **Soft-confirmed** by the sequencer (~instant)
2. **Committed** to Bitcoin via sequencer commitment inscription
3. **ZK-proven** on Bitcoin via batch proof inscription
4. **Bridge-secured** by BitVM2 fraud proofs (for any BTC ↔ cBTC flows)

This means a completed BINST process instance has its execution
integrity guaranteed by ZK math on Bitcoin — and any BTC liquidity
involved is secured by BitVM2's fraud-proof mechanism. The institution's
identity (inscription) and the execution proof (ZK batch) both live on
Bitcoin L1.

## Why This Matters for BINST

Process instance state reaches Bitcoin via batch proof. This means:

- A second L2 can verify process execution by **reading Bitcoin DA** — no messaging protocol needed
- The `binst-protocol` CLI can reconstruct which steps were executed, by whom, at what timestamp
- Cross-chain execution verification is **trustless** — it goes through Bitcoin, not through any relay
- The Execute view shows real-time finality progression for every transaction

See: [Citrea DA Format](./citrea-da-format.md) · [Decoding Procedure](./decoding-procedure.md)
