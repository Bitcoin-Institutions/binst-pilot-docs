# Institution Anchoring Lifecycle

An institution progresses through three anchoring states:

## The Three States

```text
State 1: UNANCHORED (L2-only)
  → Process instances exist on Citrea (or any L2)
  → Functional on L2: step execution works
  → Batch proofs reach Bitcoin DA (orphan proofs — see below)
  → No inscription, no rune
  → Status: DRAFT

State 2: BINDING (partially anchored)
  → L2 instances exist + institution inscription created
  → But not yet fully linked (inscription not referenced
    in L2 instances, or rune not yet etched)
  → Status: BINDING

State 3: ANCHORED (fully sovereign)
  → Inscription created + L2 instances reference it via
    templateInscriptionId + rune etched
  → L2 state is provably linked to Bitcoin identity
  → Status: ANCHORED
```

## Design Decision: Progressive Anchoring

**Anchoring is not inscription.** An institution's processes can execute on L2 from the moment the first `BINSTProcess` instance is deployed. The institution becomes *Bitcoin-anchored* when its Ordinals inscription is created and its L2 instances reference it via `templateInscriptionId`.

This is a deliberate design choice:

- **Lowers the barrier to entry** — users can experiment on L2 for just gas costs
- **Creates a natural funnel** — experiment → anchor → grow
- **Matches reality** — Citrea batches state regardless of inscription status
- **Permissionless** — no gatekeeping on who can create institutions

## Orphan ZK Proofs

If a user creates L2 process instances on Citrea but never inscribes the institution identity on Bitcoin, the ZK batch proof still reaches Bitcoin DA. This is an **orphan proof** — valid (it proves the L2 state transition happened) but unanchored (no Bitcoin-native identity to attach it to).

The `binst-decoder` would see storage slot changes for BINSTProcess instances, but no `templateInscriptionId` would resolve to a valid Bitcoin inscription.

Orphan proofs are **not harmful**:

- They are noise the indexer filters: `if templateInscriptionId resolves to nothing → skip`
- They don't affect anchored institutions
- They represent experimentation — a healthy signal for protocol adoption

## Entity Creation Patterns

| Pattern | Bitcoin TX needed? | L2 TX needed? | Example |
|---|---|---|---|
| **Full entity creation** | Yes (inscription + rune) | Yes (create instance) | Anchored institution + process |
| **L2-only creation** | No | Yes (create instance) | Unanchored process execution |
| **Step execution** | No | Yes (EVM tx) | Execute step in BINSTProcess |
| **Verification** | No | No (read only) | Check membership, verify proof |
