# Vault Unlock Flows

The vault locks the inscription UTXO but does not make it permanently frozen.

## Path A — Admin Transfer (Leaf 0, ~24h delay)

```text
1. Admin decides to move the inscription (transfer, reinscribe, rotate key)

2. Admin waits until the UTXO is at least 144 blocks old

3. Admin constructs a Bitcoin transaction:
   - Input: the vault UTXO (nSequence = 144)
   - Witness: <admin_signature> <leaf_0_script> <control_block>
   - Output 0: new destination (fresh vault or new owner's vault)

4. Bitcoin consensus validates:
   a) admin_signature valid for admin_pubkey  → OP_CHECKSIG ✓
   b) nSequence ≥ 144 blocks passed           → OP_CSV ✓
   c) Taproot script-path commitment correct   → control_block ✓

5. Transaction confirms. Inscription sat moves to new output.
```

## Path B — Committee Override (Leaf 1, immediate)

```text
1. Emergency: admin key lost or compromised

2. Two of three committee members co-sign

3. Committee constructs a transaction:
   - Input: the vault UTXO (nSequence = 0)
   - Witness: <sig_C_or_empty> <sig_B_or_empty> <sig_A> <leaf_1_script> <control_block>
   - Output 0: recovery destination

4. Bitcoin consensus validates:
   a) 2-of-3 via OP_CHECKSIGADD    → accumulated count ≥ 2 ✓
   b) Script-path commitment        → control_block ✓

5. Transaction confirms immediately.
```

## Re-locking After a Spend

Each vault-to-vault transfer resets the CSV timer:

```text
Vault A ──(admin spends after CSV)──▶ Vault B ──(...)──▶ Vault C
```

## When Would the Admin Unlock?

| Scenario | Path | What happens next |
|---|---|---|
| Transfer to new admin | Leaf 0 | Send to new admin's vault; call `transferAdmin()` on L2 |
| Rotate admin key | Leaf 0 | Send to vault with new admin pubkey |
| Reinscribe (update metadata) | Leaf 0 | Spend → new reveal TX → re-vault |
| Admin key compromised | Leaf 1 | Committee moves to safe address |
| Admin key lost | Leaf 1 | Committee recovers to new admin's vault |
| Migrate to covenant vault | Leaf 0 | Move to OP_CTV vault when available |

## Atomic Institution Transfer

An institution accumulates vault UTXOs over time — the institution
inscription itself plus child inscriptions (process templates).
Transferring admin means moving **all** of them.

Atomicity comes from Bitcoin itself: any transaction with multiple
inputs and multiple outputs is all-or-nothing by consensus rules.
A single transaction spending every vault UTXO guarantees that either
all inscriptions move to the new admin or none do:

```text
Single transaction (all-or-nothing):

  Inputs:                              Outputs:
  ─────────                            ────────
  vault UTXO 1 (Institution sat)  →   new vault 1 → new admin's key
  vault UTXO 2 (Template A sat)   →   new vault 2 → new admin's key
  vault UTXO 3 (Template B sat)   →   new vault 3 → new admin's key
  fee input (admin's spending)    →   change → admin
```

Each input uses the Leaf 0 script-path spend. The new admin verifies
that every inscription is routed to the correct new vault before the
transaction is broadcast.

### CSV maturity constraint

Every vault UTXO must be ≥ 144 blocks old for Leaf 0 spends. If a
child inscription was created recently (e.g., a new process template),
its vault may not have matured yet. In that case, the transfer is
split into two transactions:

- **TX 1:** all matured vault UTXOs (transfers immediately)
- **TX 2:** remaining UTXOs (transfers once CSV matures)

### Committee recovery

If the admin is uncooperative, the committee (Leaf 1, 2-of-3 multisig)
can construct the same multi-input transaction. Leaf 1 has no CSV
delay — committee transfers are immediate.

### PSBT as a signing tool

A PSBT (BIP-174/371) is not required for atomicity — any Bitcoin
transaction is inherently atomic. PSBT is a **workflow format**: an
ephemeral container for building, inspecting, and co-signing a
transaction before broadcast. It is recommended when:

- **Cold keys** — the admin signs offline and passes the file to
  a watch-only node for broadcast.
- **Committee spends** — two members sign independently, combine
  signatures, then finalize — without ever being online at the
  same time.
- **Audit before broadcast** — any party can decode the PSBT and
  verify inputs, outputs, and script paths before a signature is
  added.

Once broadcast, the PSBT ceases to exist. The vaults and their
Tapscript leaves are what protect the UTXOs — PSBT is just the
recommended tool for spending from them.
