# Process Execution

Processes are the core operational primitive of BINST — multi-step workflows that run on L2.

## Concepts

- **ProcessTemplate** — an immutable blueprint defining a sequence of steps (name, description, action type, configuration)
- **ProcessInstance** — a running execution of a template, tracking which steps have been completed, by whom, and when

## Executing a Step

```text
1. Member calls executeStep() on L2 ProcessInstance
   → complex validation, payment, state transitions happen on L2
   → event emitted: StepExecuted(who, stepIndex, timestamp)

2. (Optional) Member inscribes step execution as child of instance
   → permanent, discoverable record on Bitcoin
   → not required for protocol correctness (batch proof handles that)
   → makes it human-readable on explorers

3. L2 batch proof writes state diff to Bitcoin
   → step execution is ZK-proven
```

## Entity-to-Primitive Mapping

| Entity | Nature | Bitcoin Primitive | Reasoning |
|---|---|---|---|
| **Institution** | Unique, one-of-one | Ordinal inscription | Needs metadata, provenance, UTXO-based ownership |
| **Process Template** | Unique, immutable | Ordinal inscription (child of institution) | Unique artifact with structured content |
| **Process Instance** | Unique, mutable state | Ordinal inscription (child of template) | State updates via batch proofs |
| **Step Execution** | Immutable event record | Ordinal inscription (child of instance) | Permanent discoverable record |
| **Membership** | Fungible relationship | Rune balance | "Hold ≥1 token = member" |
| **Governance vote** | Fungible weight | Rune balance (separate) | Transferable, weighted voting power |

## Cross-Chain Execution Safety

A `ProcessInstance` has exactly one **home chain**. Steps execute only on that chain. This is enforced by design:

- Mirror chains provide **read-only** identity and membership verification
- Mirror contracts (`InstitutionMirror.sol`) have no `executeStep()` function
- The type system prevents concurrent mutation across chains
- No rollback/rewind mechanism is needed — conflicts are prevented, not repaired

If a process on one L2 needs to *reference* a step completed on another L2, it performs a **cross-chain read** (via Bitcoin DA batch proof or LayerZero query). No mutation, no conflict.

See [Cross-Chain Synchronization](./cross-chain.md) for the full model.
