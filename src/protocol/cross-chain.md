# Cross-Chain Synchronization

> **⚠ Architectural vision — not built in the pilot.**
> This page describes Phase 3 design. The current pilot runs on a single L2 (Citrea). Cross-chain relay and mirror contracts do not exist yet. The Bitcoin DA verification path is live; the LayerZero identity sync is not.

BINST institutions can be **omnipresent across multiple L2s** simultaneously, not just portable between them.

## The Dual-Channel Model

```text
Bitcoin (inscription + rune) = AUTHORITY
    ↓
Home L2 (e.g., Citrea) = PRIMARY DELEGATE (read + write)
    ↓ LayerZero V2 (identity)     ↓ Bitcoin DA (execution proof)
Mirror L2s (BOB, Rootstock, etc.) = READ-ONLY MIRRORS
```

Two sync channels, each optimized for different needs:

| Channel | What it syncs | Speed | Trust model |
|---|---|---|---|
| **LayerZero V2** | Identity: name, admin, inscriptionId | Fast (real-time) | DVN-configurable |
| **Bitcoin DA** | Execution: process step states, completion proofs | Slow (batch interval) | Trustless (ZK-proven) |

## LayerZero V2 on Citrea

LayerZero V2 has deployed endpoints on **Citrea mainnet** (Chain ID 4114, Endpoint ID 30403, endpoint address `0x6F475642a6e85809B1c36Fa62763669b1b48DD5B`) and supports 8+ Bitcoin L2s:

- Citrea, BOB, Bitlayer, BEVM, Merlin, Rootstock, Hemi, Corn, Goat

This maps directly to BINST's L2 portability promise.

## The Three State Tiers

| Tier | Data | Sync method | Writable? |
|---|---|---|---|
| **Identity** | inscriptionId, admin, name | LayerZero (fast) | Home chain only |
| **Membership** | Rune balance | Bitcoin L1 (authoritative) | Bitcoin only |
| **Execution** | stepStates[], process progress | Bitcoin DA (trustless) | Home chain only |

## Single-Writer Rule

**Critical invariant:** A `BINSTProcess` instance lives on exactly one home chain. It is created there, executed there, completed there. Mirror chains can *read* process state but cannot *mutate* it.

This prevents the core distributed systems problem: two L2s executing the same step simultaneously and producing conflicting state.

### Why not rewind/rollback?

A rollback mechanism would mean:
1. You allowed the conflict to happen
2. You detected it after the fact
3. You unwound one or both executions

This adds enormous complexity and violates the simplicity principle. Instead, **architectural prevention** eliminates the problem entirely:

- Mirror contracts expose only view functions: read-only access to process state, step status, and completion
- No `executeStep()`, no `createInstance()`, no mutating functions
- The type system enforces the invariant at compile time

### Cross-chain process references

If a process on BOB needs to verify a step completed on Citrea (e.g., "KYC must complete before this audit"), it performs a **cross-chain read**:

- Query the Citrea mirror via LayerZero (fast, real-time)
- Or verify the step state from Bitcoin DA batch proof (slower, trustless)

No mutation, no conflict.

## Trust Considerations

LayerZero introduces a dependency outside Bitcoin — messages go through DVNs (Decentralized Verifier Networks), not Bitcoin DA. This is acceptable because:

- The **authority** remains on Bitcoin (inscription UTXO ownership)
- Mirrors are **convenience, not consensus** — any L2 can independently verify the inscription on Bitcoin
- If LayerZero goes down, institutions still function on their home L2
- LayerZero's principles align with BINST: permissionless, immutable endpoints, censorship-resistant

## Implementation Plan

Phase 3 will introduce:

- **BINSTRelay** — an OApp that listens for institution events on the home L2 and broadcasts identity state to registered mirror chains
- **ProcessMirror** — a read-only contract on non-home chains that receives and exposes process state and institution identity data

This turns BINST from "portable across L2s" (manual redeploy) into "**omnipresent across L2s**" (automatic sync).
