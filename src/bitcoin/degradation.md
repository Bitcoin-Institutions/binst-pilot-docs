# Graceful Degradation

The protocol degrades hierarchically — the closer to Bitcoin you lose, the harder the recovery.

## Losing the L2 (Graceful)

If Citrea goes down or the user wants to switch L2:

1. Inscription **data** is permanent and readable on Bitcoin forever
2. Membership Runes **continue to function** on Bitcoin L1
3. Admin **creates new process instances** on another L2
4. New instances reference the **same inscription ID** via `templateInscriptionId`
5. Institution continues with full identity and membership intact

## Losing the Inscription UTXO (Serious)

If the admin accidentally spends the inscription UTXO despite vault protection:

1. Inscription **data** is permanent and readable forever
2. L2 process instances **continue to function** short-term
3. Admin **re-inscribes** a recovery record (child of original)
4. New L2 instances reference the **new inscription ID**
5. Original provenance chain is **preserved**

The vault script exists specifically to make this scenario extremely unlikely.

## Losing the Bitcoin Key (Catastrophic)

1. Committee (Leaf 1, 2-of-3 multisig) **recovers the inscription** to a new key's vault
2. Admin **creates new L2 process instances** from the new key
3. This is the hardest recovery — the committee backstop is the last resort

## The Hierarchy

```text
L2 down              → create instances elsewhere, identity survives on Bitcoin
Inscription UTXO lost → re-inscribe + new L2 instances (harder, but recoverable)
Bitcoin key lost      → committee recovery (hardest, requires multi-sig)
```
