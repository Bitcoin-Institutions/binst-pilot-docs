# Smart Contracts

## Architecture: Thin L2, Fat L1

BINST follows a **"Thin L2, Fat L1"** architecture. Bitcoin L1 is the
source of truth for identity (institutions) and definitions (process
templates). The L2 only holds **execution state**.

Two contracts make up the L2 layer:

| Contract | Purpose |
|---|---|
| `BINSTProcessFactory` | Thin factory — deploys process instances, indexes by user and template |
| `BINSTProcess` | Self-contained instance — embedded steps, L1 anchor, migration-ready |

Bitcoin owns identity and definitions; L2 only owns execution.

---

## Architecture

### BINSTProcessFactory

A thin factory deployed **once per L2 chain**. Creates self-contained
process instances and maintains indexing by user and by L1 template.

**Deployed addresses:**

| Network | Address | Chain ID |
|---|---|---|
| Citrea testnet | `0x6a1d2adbac8682773ed6700d2118c709c8ce5000` | 5115 |
| Hardhat local | `0x9fe46736679d2d9a65f0992f2272de9f3c7fa6e0` | 31337 |

**Key functions:**

| Function | Selector | Purpose |
|---|---|---|
| `createInstance(string, string[], string[])` | `0x6f794b70` | Deploy a new `BINSTProcess` instance |
| `getTemplateInstances(string)` | `0xb43bed00` | All instances for a given L1 template inscription ID |
| `getUserInstances(address)` | `0xfceaae17` | All instances created by an address |
| `getInstanceCount()` | `0xae34325c` | Total instance count |
| `allInstances(uint256)` | `0x9b0dc489` | Instance address by global index |

**Events:**

| Event | Topic0 |
|---|---|
| `InstanceCreated(address indexed, address indexed, string, uint256)` | `0xa3de24f67beb…0700f` |

### BINSTProcess

A **self-contained** process instance anchored to a Bitcoin L1
inscription. Each instance carries its own step definitions — no
dependency on separate contracts on L2.

**Why self-contained?** Cross-chain migration. When a partially-executed
process migrates from Citrea to another L2 (via LayerZero), the
receiving chain only needs one message:

```
{templateInscriptionId, steps[], currentStepIndex, stepStates[], creator}
```

No pre-deployed contracts are required on the destination chain.

**Provenance chain (L2 → L1):**
```text
BINSTProcess.templateInscriptionId
  → look up on Bitcoin (mempool.space / ord indexer)
  → read inscription body → institution_id, steps, admin pubkey
  → verify the L1 hierarchy: institution → template → (this instance)
```

**Key state:**

```solidity
string public templateInscriptionId;   // L1 anchor — Bitcoin inscription ID
Step[] public steps;                    // Step definitions (name + actionType)
address public creator;                 // Who created this instance
uint256 public currentStepIndex;        // Current step (0-based)
bool public completed;                  // True when all steps done
mapping(uint256 => StepState) public stepStates;

// Migration fields (future — Phase 65)
uint32 public sourceChainId;           // 0 if native, non-zero if migrated
address public sourceAddress;          // Address on source chain
```

**Key functions:**

| Function | Selector | Purpose |
|---|---|---|
| `executeStep(uint8, string)` | `0xf16e3a23` | Execute the current step with status + evidence |
| `currentStepIndex()` | `0x334f45ec` | Read current step index |
| `completed()` | `0x9d9a7fe9` | Check if all steps are done |
| `totalSteps()` | `0x6931b3ae` | Total number of steps |
| `creator()` | `0x02d05d3f` | Address of the instance creator |
| `templateInscriptionId()` | `0x0270a0b3` | The L1 inscription this instance is anchored to |

**Step status enum:** `0 = Pending`, `1 = Completed`, `2 = Rejected`

**Events:**

| Event | Topic0 |
|---|---|
| `StepExecuted(uint256 indexed, address indexed, uint8, string)` | `0x5b89aea5…1b82` |
| `ProcessCompleted(address indexed, uint256)` | *(keccak256 of signature)* |

---

## Instance Lifecycle

```text
1. Factory deployed once per L2 chain
   └─ Citrea testnet: 0x6a1d…5000

2. User creates instance from the Execute view
   └─ factory.createInstance(inscriptionId, stepNames[], stepActionTypes[])
   └─ InstanceCreated event → instance address parsed from receipt
   └─ Instance address saved to localStorage and shown in the UI

3. Steps executed sequentially
   └─ instance.executeStep(Completed, evidenceData)
   └─ StepExecuted event emitted per step
   └─ currentStepIndex advances; UI reloads state from contract

4. Process completes
   └─ instance.completed == true after final step
   └─ ProcessCompleted event emitted

5. Bitcoin settlement (automatic — no user action)
   └─ Soft Confirmation: sequencer signs the L2 block (~instant)
   └─ Committed: sequencer commitment inscribed on Bitcoin
   └─ Proven: ZK batch proof inscribed on Bitcoin
   └─ All L2 state is now part of the Bitcoin permanent record

6. Migration (future — Phase 65)
   └─ LayerZero _lzSend() carries full state to destination L2
   └─ Destination deploys new BINSTProcess pre-loaded with state
```

### Bitcoin Settlement Finality

Every L2 transaction goes through three finality stages before reaching
Bitcoin permanence:

| Stage | Meaning | Verification |
|---|---|---|
| **Soft Confirmation** | Sequencer has ordered the tx in an L2 block | Transaction receipt |
| **Committed** | Sequencer commitment inscribed on Bitcoin | `citrea_getLastCommittedL2Height` |
| **Proven** | ZK proof of correct execution inscribed on Bitcoin | `citrea_getLastProvenL2Height` |

The webapp tracks these stages per-transaction (instance creation and
each step execution) and displays finality badges in the Execute view.
See [ZK Batch Proofs](../bitcoin/batch-proofs.md) for the full model.

---

## EVM Configuration

Citrea does not support Cancun opcodes. Target **Shanghai**:

```typescript
solidity: {
  version: "0.8.24",
  settings: { evmVersion: "shanghai", optimizer: { enabled: true, runs: 200 } }
}
```

## Solidity Tests

24 tests in `test/BINSTPilot.ts` (Hardhat 3, `node:test`):
- 10 active contract tests (BINSTProcessFactory, BINSTProcess)
- 14 additional tests covering extended contract lifecycles
