# Graceful Degradation

The protocol degrades hierarchically — the closer to Bitcoin you lose, the harder the recovery.

## Losing the L2 Contract (Graceful)

If Citrea goes down or the user wants to switch L2:

1. Inscription **data** is permanent and readable on Bitcoin forever
2. Membership Runes **continue to function** on Bitcoin L1
3. Admin **deploys a new contract** on another L2
4. New contract references the **same inscription ID** and rune ID
5. Institution continues with full identity and membership intact

## Losing the Inscription UTXO (Serious)

If the admin accidentally spends the inscription UTXO despite vault protection:

1. Inscription **data** is permanent and readable forever
2. L2 contracts **continue to function** short-term
3. Admin **re-inscribes** a recovery record (child of original)
4. L2 contracts are **updated** to reference the new inscription
5. Original provenance chain is **preserved**

The vault script exists specifically to make this scenario extremely unlikely.

## Losing the Bitcoin Key (Catastrophic)

1. Committee (Leaf 1, 2-of-3 multisig) **recovers the inscription** to a new key's vault
2. Admin **deploys new L2 contracts** from the new key
3. This is the hardest recovery — the committee backstop is the last resort

## The Hierarchy

```text
L2 contract lost     → redeploy elsewhere, identity survives on Bitcoin
Inscription UTXO lost → re-inscribe + update L2 (harder, but recoverable)
Bitcoin key lost      → committee recovery (hardest, requires multi-sig)
```
