# Switching L2s

One of BINST's core properties: the L2 is replaceable. Here's how migration works.

## Flow

```text
1. Admin decides to move from Citrea to another L2 (e.g., BOB)

2. BINSTProcess instance carries full state:
   → templateInscriptionId (L1 anchor — same on any chain)
   → steps[] (embedded at creation, no external dependency)
   → currentStepIndex + stepStates[] (current execution state)
   → creator address

3. Migration via LayerZero (Phase 65 — future):
   → _lzSend() on source chain carries the full state
   → Destination chain deploys new BINSTProcess with that state
   → sourceChainId + sourceAddress fields link back to origin

4. The Bitcoin-layer identity is unchanged:
   → same inscription, same UTXO, same admin key
   → same membership Rune, same member balances
   → provenance chain is intact

5. The old L2 instance becomes historical — its batch proofs
   remain on Bitcoin as a permanent record of past operations

6. The new L2 instance continues execution from where it left off
```

## What Survives

| Element | After migration |
|---|---|
| Inscription ID | ✅ Unchanged — same identity on Bitcoin |
| Admin key | ✅ Unchanged — same UTXO, same authority |
| Membership Runes | ✅ Unchanged — live on Bitcoin, not on any L2 |
| Provenance chain | ✅ Unchanged — parent/child inscriptions intact |
| Old L2 state | ✅ Preserved — batch proofs on Bitcoin are permanent |
| New L2 instance | 🆕 Deployed with full state, same `templateInscriptionId` |

## Why This Works

The Bitcoin key is the root of authority, not the L2 contract address.
The inscription is the institution's identity, not the Solidity code.
When you move L2s, you're changing the **processing engine**, not the
institution itself.

Because `BINSTProcess` instances are **self-contained** (embedded step
definitions, no dependency on external L2 contracts), migration is a
single message. The destination chain doesn't need a pre-deployed
factory, institution, or template — it just deploys a new `BINSTProcess`
pre-loaded with the migrated state.
