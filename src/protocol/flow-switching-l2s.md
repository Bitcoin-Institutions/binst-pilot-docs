# Switching L2s

One of BINST's core properties: the L2 is replaceable. Here's how migration works.

## Flow

```text
1. Admin decides to move from Citrea to another L2 (e.g., BOB)

2. Admin deploys new Institution contract on the new L2
   → binds it to the SAME inscription ID and rune ID

3. The Bitcoin-layer identity is unchanged:
   → same inscription, same UTXO, same admin key
   → same membership Rune, same member balances
   → provenance chain is intact

4. The old L2 contract becomes historical — its batch proofs
   remain on Bitcoin as a permanent record of past operations

5. New operations flow through the new L2 contract
   → the institution continues seamlessly
```

## What Survives

| Element | After migration |
|---|---|
| Inscription ID | ✅ Unchanged — same identity on Bitcoin |
| Admin key | ✅ Unchanged — same UTXO, same authority |
| Membership Runes | ✅ Unchanged — live on Bitcoin, not on any L2 |
| Provenance chain | ✅ Unchanged — parent/child inscriptions intact |
| Old L2 state | ✅ Preserved — batch proofs on Bitcoin are permanent |
| New L2 contract | 🆕 New address, bound to same inscription |

## Why This Works

The Bitcoin key is the root of authority, not the L2 contract address. The inscription is the institution's identity, not the Solidity code. When you move L2s, you're changing the **processing engine**, not the institution itself.

This is analogous to moving a company's operations from one country to another — the company's identity (registration, brand, ownership) doesn't change. Only the operational jurisdiction does.
