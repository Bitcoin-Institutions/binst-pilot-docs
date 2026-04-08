# Process Execution

Processes are the core operational primitive of BINST — multi-step workflows that run on L2.

## Concepts

- **ProcessTemplate** — an immutable blueprint defining a sequence of steps (name, description, action type, configuration)
- **ProcessInstance** — a running execution of a template, tracking which steps have been completed, by whom, and when

## Executing a Step

```text
1. Member clicks "Execute Step" in the webapp Execute view
   → StepExecution JSON is built from active step context
   → L2Action (ExecuteStep) is pushed to the L2 queue
   → Stack panel opens automatically on the L2 tab

2. Member reviews the L2 queue, clicks "Send Next"
   → eth_sendTransaction is called via MetaMask / SafePal EVM
   → One wallet pop-up per action (EVM calls are not batchable)
   → executeStep() fires on the ProcessInstance contract

3. L2 emits StepExecuted(who, stepIndex, timestamp)
   → Step state is updated in the ProcessInstance

4. (Optional) Member inscribes step execution as a child inscription on L1
   → Push StepExecution entity to the L1 stack
   → Build PSBT via UniSat / SafePal BTC
   → Permanent, human-readable record on Bitcoin

5. L2 batch proof writes state diff to Bitcoin
   → Step execution is ZK-proven on L1
```

> **Signing model:** Step execution involves two independent signing
> flows. The EVM step call is signed by the **EVM wallet** (MetaMask or
> SafePal). The optional L1 inscription is signed by the **Bitcoin
> wallet** (UniSat or SafePal BTC) via PSBT. The webapp maintains
> separate wallet slots for each layer. See
> [WASM Webapp](../implementation/webapp.md) for the full signing model.

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

- Mirror chains provide **read-only** identity and membership verification *(Phase 3 plan)*
- Mirror contracts (`InstitutionMirror.sol`) would have no `executeStep()` function — write operations are structurally excluded
- The type system prevents concurrent mutation across chains
- No rollback/rewind mechanism is needed — conflicts are prevented, not repaired

If a process on one L2 needs to *reference* a step completed on another L2, it performs a **cross-chain read** (via Bitcoin DA batch proof or LayerZero query). No mutation, no conflict.

See [Cross-Chain Synchronization](./cross-chain.md) for the full model.
