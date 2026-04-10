# The Three-Layer Model

## Layer 1: Bitcoin (Authority)

Bitcoin L1 stores three kinds of data:

| Primitive | Role | What it represents |
|---|---|---|
| **Ordinals inscriptions** | Entity identity, ownership, metadata | Institutions, process templates, process instances |
| **Runes** | Membership and fungible roles | "Alice is a member of Acme Financial" |
| **ZK batch proofs** | Computational integrity | Every L2 state transition, ZK-proven |

```text
Bitcoin L1
├── Ordinals    → entities EXIST here (identity, ownership — AUTHORITATIVE)
├── Runes       → membership IS here (fungible tokens per institution)
└── ZK proofs   → computation is PROVEN here (L2 batch proofs)
```

This is the **authoritative layer**. If there's a conflict between Bitcoin and any L2, Bitcoin wins.

## Layer 2: Processing Delegate (Currently Citrea)

The L2 runs a **minimal execution layer** — only process instance state.
Identity (institutions) and definitions (process templates) live on
Bitcoin as inscriptions. The L2 contracts are:

- **`BINSTProcessFactory`** — thin factory, deployed once per chain,
  creates self-contained `BINSTProcess` instances
- **`BINSTProcess`** — carries its own step definitions +
  `templateInscriptionId` anchor back to Bitcoin L1

Each instance is **self-contained**: it embeds the step names and action
types copied from the L1 template at creation time. This means an
instance can migrate to another L2 without any pre-deployed contracts on
the destination.

The L2 is a processing engine — it does not own the identity. The user
can redeploy to a different L2 at any time, pointing the new instance at
the same `templateInscriptionId`. The identity stays on Bitcoin.

### Why Citrea?

| Feature | Why it matters |
|---|---|
| Fully EVM-compatible | Solidity process instances deploy with an RPC endpoint change |
| Bitcoin Light Client (`0x3100…0001`) | Read Bitcoin block hashes on-chain, verify inclusion proofs |
| Schnorr precompile (`0x…0200`) | BIP-340 signature verification in Solidity — no other L2 offers this |
| Clementine Bridge (BitVM2) | Trust-minimized BTC ↔ cBTC peg |
| Testnet uses Bitcoin Testnet4 as DA | Real Bitcoin data, not simulated |
| Three finality levels | Soft confirmation → Committed → ZK-proven on Bitcoin |

The L2 choice is explicitly **non-permanent**. The architecture allows migrating to any EVM-compatible L2 by deploying a new factory and creating instances that reference the same inscription IDs.

## Layer 3: Verification

The verification layer provides trust-minimized guarantees that L2
computation was correct. On Citrea, **two mechanisms are already
operational:**

### ZK Batch Proofs (operational)

Citrea's batch prover inscribes ZK proofs (Groth16 via RISC Zero) on
Bitcoin. Every L2 state transition — including every `executeStep()` call
on a `BINSTProcess` — is covered by these proofs. The webapp tracks
three finality stages per transaction:

| Stage | Meaning |
|---|---|
| **Soft Confirmation** | Sequencer has ordered the tx |
| **Committed** | Sequencer commitment inscribed on Bitcoin |
| **Proven** | ZK proof inscribed on Bitcoin — mathematically verified |

### BitVM2 / Clementine Bridge (operational)

Citrea's **Clementine Bridge** implements **BitVM2** for trust-minimized
BTC ↔ cBTC peg verification. BitVM2 is a fraud-proof system where:

- **Optimistic case:** bridge operators post assertions about L2 state;
  if unchallenged, the assertion is accepted
- **Dispute case:** a challenger can force the operator to reveal
  intermediate computation steps; if the operator is wrong, their bond
  is slashed on Bitcoin L1
- **Trust assumption:** 1-of-N honesty — only one honest verifier needed

This means BINST process execution on Citrea is not just proven by ZK
proofs — the bridge itself that connects BTC liquidity to the L2 is
secured by BitVM2 fraud proofs on Bitcoin.

### Future enhancements

As Bitcoin's scripting capabilities evolve:

- **Covenants** (OP_CTV, OP_CAT) — native spending constraints for vaults and trustless bridges
- **On-chain SNARK verification** — ZK proof verification within Bitcoin Script
- **BitVM3 / BitVMX** — further improvements to fraud-proof and RISC-V verification

## What Each Layer Guarantees

| Layer | What it proves | Trust assumption | Failure mode |
|---|---|---|---|
| **Ordinal inscription** | Entity exists, admin controls UTXO | Bitcoin consensus | UTXO accidentally spent → lose root authority |
| **Rune balance** | This person is a member | Bitcoin consensus | Token accidentally sent → membership lost |
| **L2 process instance** | Processing delegate executes logic | Bitcoin consensus + ZK math | L2 down → create instances on another L2 |
| **L2 batch proof** | Every state transition was correct | Bitcoin consensus + ZK math | Proof missing → state unverifiable until next batch |

**The Bitcoin key is the single root of authority.** L2 process instances are replaceable processing delegates. Losing an L2 is graceful — create new instances elsewhere. Losing the Bitcoin key is catastrophic — the committee multi-sig is the last resort.
