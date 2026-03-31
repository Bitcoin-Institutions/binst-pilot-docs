# Smart Contracts

Four Solidity contracts deployed and verified on Citrea Testnet (Chain 5115, Shanghai EVM, Solidity 0.8.24).

## Contract Overview

| Contract | Purpose |
|---|---|
| `BINSTDeployer` | Factory/registry — creates institutions and deploys process templates |
| `Institution` | Institution entity — members, admin, Bitcoin identity (`inscriptionId`, `runeId`) |
| `ProcessTemplate` | Immutable workflow blueprint with named steps |
| `ProcessInstance` | Running execution with step-by-step state tracking |

## BINSTDeployer

The factory and registry for all BINST entities. Creates institutions and process templates, maintains a global registry.

**Key functions:**
- `createInstitution(name, admin)` → deploys a new `Institution` contract
- `createProcessTemplate(institutionAddr, steps[])` → deploys an immutable template
- `createProcessInstance(templateAddr)` → creates a running instance

## Institution

Defines an institution with Bitcoin anchoring.

**State:**
- `name` — institution name
- `admin` — current admin address (EVM)
- `inscriptionId` — Ordinals inscription ID (Bitcoin anchor)
- `runeId` — Rune ID for membership
- `members[]` — member addresses
- `isMember` mapping — O(1) membership check
- `processes[]` — deployed process contracts

**Key functions:**
- `setInscriptionId(id)` / `setRuneId(id)` — bind to Bitcoin identity
- `addMember(addr)` / `removeMember(addr)` — manage membership
- `transferAdmin(newAdmin)` — transfer control

## ProcessTemplate

Immutable blueprint defining a sequence of steps.

**Step struct:**
```solidity
struct Step {
    string name;
    string description;
    string actionType;
    string config;
}
```

Steps are set at deployment and cannot be modified — the template is a permanent record.

## ProcessInstance

A running execution of a template, tracking step-by-step progress.

**StepState struct:**
```solidity
struct StepState {
    StepStatus status;   // Pending, Completed, Skipped
    address actor;       // who executed it
    string data;         // execution data
    uint256 timestamp;   // when it was executed
}
```

**Key function:**
- `executeStep(stepIndex, data)` — complete a step (only callable once per step)

## EVM Configuration

Citrea does not support Cancun. Target **Shanghai**:

```typescript
solidity: {
  version: "0.8.24",
  settings: { evmVersion: "shanghai", optimizer: { enabled: true, runs: 200 } }
}
```
