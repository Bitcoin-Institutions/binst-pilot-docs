# Admin Transfer

Transferring institutional control from one admin to another.

## Flow

```text
1. Current admin transfers the inscription UTXO to new admin's vault
   → on Bitcoin: new admin now controls the UTXO (Leaf 0 spend)
   → the inscription ID stays the same; the controlling key changes

2. New admin calls transferAdmin() on the L2 contract
   → L2 contract updates admin address to match new key holder

3. Both layers now agree: the new admin controls the institution
   on Bitcoin (UTXO) and on the L2 (contract state)
```

## Conflict Resolution

If the L2 contract admin disagrees with the UTXO owner, **the UTXO owner is authoritative**. The L2 contract is expected to be updated to match.

A future version could enforce this via Bitcoin-key-based signature verification on the L2 (using Citrea's Schnorr precompile).

## Security Considerations

The Taproot vault provides safety for admin transfers:

- **Leaf 0 (admin)**: CSV-delayed (~24 hours / 144 blocks). The delay gives time to abort if the transfer was unauthorized.
- **Leaf 1 (committee)**: Immediate 2-of-3 multisig. Emergency override if the admin key is compromised during transfer.

See [Taproot Vault](../bitcoin/taproot-vault.md) for the full vault specification.
