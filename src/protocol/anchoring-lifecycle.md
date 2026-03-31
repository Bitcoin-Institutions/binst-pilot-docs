# Institution Anchoring Lifecycle

An institution progresses through three anchoring states:

## The Three States

```text
State 1: UNANCHORED (L2-only)
  → Contract deployed on Citrea (or any L2)
  → Functional on L2: members, processes work
  → Batch proofs reach Bitcoin DA (orphan proofs — see below)
  → No inscription, no rune
  → Status: DRAFT

State 2: BINDING (partially anchored)
  → Contract deployed + inscription created
  → But not yet fully linked (setInscriptionId not called,
    or rune not yet etched)
  → Status: BINDING

State 3: ANCHORED (fully sovereign)
  → Contract deployed + inscription bound + rune etched + rune bound
  → L2 state is provably linked to Bitcoin identity
  → Status: ANCHORED
```

## Design Decision: Progressive Anchoring

**Anchoring is not deployment.** An institution exists on the L2 from the moment its contract is deployed. It becomes *Bitcoin-anchored* when its Ordinals inscription is created and bound to the contract.

This is a deliberate design choice:

- **Lowers the barrier to entry** — users can experiment on L2 for just gas costs
- **Creates a natural funnel** — experiment → anchor → grow
- **Matches reality** — Citrea batches state regardless of inscription status
- **Permissionless** — no gatekeeping on who can create institutions

## Orphan ZK Proofs

If a user deploys `Institution.sol` on Citrea but never inscribes the ordinal identity, the ZK batch proof still reaches Bitcoin DA. This is an **orphan proof** — valid (it proves the L2 state transition happened) but unanchored (no Bitcoin-native identity to attach it to).

The `binst-decoder` would see storage slot changes for an Institution contract, but `inscriptionId` and `runeId` would be empty strings.

Orphan proofs are **not harmful**:

- They are noise the indexer filters with a simple check: `if inscriptionId == "" → skip`
- They don't affect anchored institutions
- They represent experimentation — a healthy signal for protocol adoption

## Entity Creation Patterns

| Pattern | Bitcoin TX needed? | L2 TX needed? | Example |
|---|---|---|---|
| **Full entity creation** | Yes (inscription + rune) | Yes (deploy + bind) | Create institution |
| **L2-only creation** | No | Yes (deploy) | Unanchored institution, process template |
| **Event registration** | No | Yes (EVM tx) | Execute step, add member |
| **Verification** | No | No (read only) | Check membership, verify proof |
