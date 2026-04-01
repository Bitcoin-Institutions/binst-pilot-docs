# Taproot Vault — UTXO Safety

The inscription UTXO is the root of authority. Losing it means losing the institution. The Taproot vault prevents accidental spending at the **consensus level**.

## The Risk

An admin who spends the inscription UTXO in a non-Ordinals-aware wallet loses control of the inscription. The data remains on Bitcoin permanently, but the UTXO tracking it moves to an unknown party.

**This is the most critical risk in the protocol.** The vault is not optional — it is essential infrastructure.

## Script Structure

```text
Taproot output:
  Internal key: NUMS point (unspendable — disables key-path spend)

  Script tree:
    Leaf 0 (admin transfer — time-delayed):
      <admin_pubkey> OP_CHECKSIG
      <144> OP_CHECKSEQUENCEVERIFY OP_DROP     ← ~24h delay

    Leaf 1 (committee override — immediate):
      <key_A> OP_CHECKSIG
      <key_B> OP_CHECKSIGADD
      <key_C> OP_CHECKSIGADD
      <2> OP_NUMEQUAL
```

## How It Works

| Path | Who | Delay | Purpose |
|---|---|---|---|
| Key path | Nobody | ∞ | Disabled (NUMS internal key) — no accidental spend possible |
| Leaf 0 | Admin (single key) | ~24 hours (144 blocks CSV) | Deliberate admin transfer with safety delay |
| Leaf 1 | 2-of-3 committee | Immediate | Emergency override for key loss or compromise |

## Why This Works

- **No accidental spending** — the key path is dead. Regular wallets can't spend it.
- **Admin retains control** — Leaf 0 allows deliberate moves with a CSV safety net.
- **Committee backstop** — Leaf 1 is the "break glass" emergency path.
- **Standard Bitcoin** — uses only Taproot (BIP 341/342), OP_CHECKSIG, OP_CSV, OP_CHECKSIGADD. No soft fork needed.
- **Ordinals-compatible** — `ord` tracks inscriptions regardless of spending script.

**Future enhancement:** When OP_CTV or OP_CAT activates, the vault can enforce that the UTXO is only spendable to pre-approved addresses (true covenant protection).

See also: [Vault Unlock Flows](./vault-unlock.md) · [Sat Isolation](./sat-isolation.md) · [Graceful Degradation](./degradation.md)
