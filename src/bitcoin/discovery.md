# Discovery

How to find and verify BINST entities — from casual browsing to full trustless verification.

## Who Needs What

| What you want to know | Where to look | Full node needed? |
|---|---|---|
| Does institution X exist? | Ordinals explorer | ❌ No |
| Who is the admin? | Inscription UTXO owner | ❌ No |
| Am I a member? | Rune balance in wallet | ❌ No |
| Who are all members? | Rune indexer query | ❌ No |
| What processes exist? | Child inscriptions | ❌ No |
| What step is instance Y on? | L2 RPC (Citrea) | ❌ No |
| Is institution X anchored? | L2 view call (`inscriptionId != ""`) | ❌ No |
| Was step execution valid? | ZK batch proof decode | ✅ Yes |
| Full trustless verification? | taproot-reader | ✅ Yes |
| Which L2 is processing? | Inscription metadata | ❌ No |
| Is this person a member? (cross-chain) | InstitutionMirror on any L2 | ❌ No |
| Did step X complete? (cross-chain) | Bitcoin DA batch proof | ✅ Yes |

## Two Tiers

```text
Tier 1 (standard tooling):  Ordinals explorer + Rune wallet
  → identity, ownership, membership — no full node needed

Tier 2 (verification):  Bitcoin full node + taproot-reader
  → ZK proof verification, state diff decoding — trustless
```

**Basic discovery requires no custom software and no full node.** Standard Ordinals and Rune tooling covers identity, ownership, membership, and provenance. The full node is only needed for the strongest verification tier.

Critically: **none of this depends on any specific L2 being online.**
