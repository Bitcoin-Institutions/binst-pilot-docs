# Process Execution

Processes are the core operational primitive of BINST — multi-step workflows
defined on Bitcoin L1 and executed on L2.

## Concepts

- **ProcessTemplate** — an immutable blueprint inscribed on Bitcoin L1, defining a sequence of steps (name, action type). A child inscription of the institution.
- **BINSTProcess** (L2) — a self-contained instance deployed on Citrea via `BINSTProcessFactory`. Carries embedded step definitions + a `templateInscriptionId` linking back to Bitcoin.

## Creating an Instance

```text
1. Admin inscribes a ProcessTemplate on Bitcoin L1 (child of institution)
   → inscription body contains step names, action types, institution_id

2. Admin opens the Execute view in the webapp
   → selects the template → clicks "Create Instance on Citrea"
   → factory.createInstance(inscriptionId, stepNames[], stepActionTypes[])
   → eth_sendTransaction via MetaMask / Brave Wallet / WalletConnect
   → InstanceCreated event → instance address parsed from tx receipt

3. Instance address is stored in localStorage and shown in the UI
```

## Executing Steps

```text
1. Member opens the Execute view, instance is loaded from Citrea
   → on-chain state read: currentStepIndex, totalSteps, completed,
     step names, step states (all via eth_call — no wallet needed)

2. Member clicks "Execute Step →"
   → instance.executeStep(Completed, evidenceData)
   → eth_sendTransaction via EVM wallet (one pop-up per step)
   → StepExecuted event emitted on-chain

3. After tx confirms (~2s on Citrea testnet):
   → UI reloads state from contract (currentStepIndex advances)
   → Tx hash saved to localStorage for finality tracking

4. Bitcoin settlement (automatic, no user action):
   → Soft Confirmation: sequencer orders the tx (~instant)
   → Committed: sequencer commitment inscribed on Bitcoin
   → Proven: ZK batch proof inscribed on Bitcoin
   → The step execution is now part of Bitcoin's permanent record
```

> **Signing model:** Each step execution requires **one EVM wallet
> signature** via `eth_sendTransaction`. There is no batching or queue —
> each step fires immediately on button click. The webapp's Execute view
> delegates all EVM/ABI logic to `citrea.rs`.

> **No L1 inscription per step:** In the current pilot, step executions
> are NOT individually inscribed on Bitcoin. They reach Bitcoin
> automatically via Citrea's ZK batch proofs. A future phase may add
> optional L1 inscriptions for step evidence.

## Entity-to-Primitive Mapping

| Entity | Nature | Bitcoin Primitive | L2 Primitive | Reasoning |
|---|---|---|---|---|
| **Institution** | Unique, one-of-one | Ordinal inscription | *(none — L1 only)* | Identity lives on Bitcoin |
| **Process Template** | Unique, immutable | Ordinal inscription (child of institution) | *(none — L1 only)* | Definition lives on Bitcoin |
| **Process Instance** | Unique, mutable state | *(represented via ZK batch proofs)* | `BINSTProcess` contract | Execution state on L2, settled to BTC via proofs |
| **Step Execution** | Immutable event record | *(settled via ZK batch proofs)* | `StepExecuted` event | Reaches Bitcoin through batch proof, not individual inscription |
| **Membership** | Fungible relationship | Rune balance | *(none — L1 only)* | "Hold ≥1 token = member" |
| **Governance vote** | Fungible weight | Rune balance (separate) | *(none — L1 only)* | Transferable, weighted voting power |

## Cross-Chain Execution Safety

A `BINSTProcess` instance has exactly one **home chain**. Steps execute only on that chain. This is enforced by design:

- Each instance is self-contained: it carries its own step definitions + `templateInscriptionId`
- No external contract dependencies on L2 — migration is a single message
- The type system prevents concurrent mutation across chains
- No rollback/rewind mechanism is needed — conflicts are prevented, not repaired

If a process on one L2 needs to *reference* a step completed on another L2, it performs a **cross-chain read** (via Bitcoin DA batch proof or LayerZero query). No mutation, no conflict.

### Migration model (future — Phase 65)

`BINSTProcess` includes `sourceChainId` and `sourceAddress` fields for LayerZero migration. The full state (`templateInscriptionId`, steps, currentStepIndex, stepStates, creator`) is sent to the destination chain, which deploys a new instance pre-loaded with that state.

See [Cross-Chain Synchronization](./cross-chain.md) and [Switching L2s](./flow-switching-l2s.md) for the full model.
