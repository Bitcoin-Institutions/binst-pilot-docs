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

Complex institutional logic executes on L2 smart contracts **as a delegate of the Bitcoin key holder**:

- Multi-step workflow execution with validation rules
- Payment processing
- Cross-contract calls and event emission
- State management (current step, completion, timestamps)

The L2 is a processing engine — it does not own the identity. The user can redeploy to a different L2 at any time, pointing the new contracts at the same inscription ID. The identity stays on Bitcoin.

### Why Citrea?

| Feature | Why it matters |
|---|---|
| Fully EVM-compatible | Solidity contracts deploy with an RPC endpoint change |
| Bitcoin Light Client (`0x3100…0001`) | Read Bitcoin block hashes on-chain, verify inclusion proofs |
| Schnorr precompile (`0x…0200`) | BIP-340 signature verification in Solidity — no other L2 offers this |
| Clementine Bridge (BitVM2) | Trust-minimized BTC ↔ cBTC peg |
| Testnet uses Bitcoin Testnet4 as DA | Real Bitcoin data, not simulated |
| Three finality levels | Soft confirmation → Committed → ZK-proven on Bitcoin |

The L2 choice is explicitly **non-permanent**. The architecture allows migrating to any EVM-compatible L2 by redeploying contracts and pointing them at the same inscription.

## Layer 3: Verification (Future)

As Bitcoin's scripting capabilities evolve, BINST will add a trust-minimized verification layer:

- **BitVM / BitVM2 / BitVM3** — fraud-proof verification of L2 state transitions
- **BitVMX** — RISC-V program execution verification on Bitcoin
- **Covenants** (OP_CTV, OP_CAT) — native spending constraints for enhanced vaults and trustless bridges
- **SNARK verification** — ZK proof verification within Bitcoin Script

This layer doesn't exist yet — it depends on Bitcoin protocol evolution. BitVM/BitVM2/BitVM3, OP_CTV, and on-chain SNARK verification are active areas of Bitcoin development and are on the BINST roadmap for future phases.

## What Each Layer Guarantees

| Layer | What it proves | Trust assumption | Failure mode |
|---|---|---|---|
| **Ordinal inscription** | Entity exists, admin controls UTXO | Bitcoin consensus | UTXO accidentally spent → lose root authority |
| **Rune balance** | This person is a member | Bitcoin consensus | Token accidentally sent → membership lost |
| **L2 contract** | Processing delegate executes logic | Bitcoin consensus + ZK math | L2 down → redeploy on another L2 |
| **L2 batch proof** | Every state transition was correct | Bitcoin consensus + ZK math | Proof missing → state unverifiable until next batch |

**The Bitcoin key is the single root of authority.** L2 contracts are replaceable processing delegates. Losing an L2 is graceful — redeploy elsewhere. Losing the Bitcoin key is catastrophic — the committee multi-sig is the last resort.
