# Sat Isolation

The inscribed satoshi lives on its own **dedicated, minimal UTXO** — separate from spending funds.

## Reveal Transaction Layout

```text
Reveal TX:
  Input 0:  commit UTXO (inscription envelope in witness)

  Output 0: 546 sats → Taproot vault address (script-guarded)
             ↑ the inscribed sat lives HERE, alone
             ├── NUMS internal key (no key-path spend)
             ├── Leaf 0: admin + 144-block CSV delay
             └── Leaf 1: 2-of-3 committee multisig

  Output 1: change → admin's regular spending wallet
             Normal sats, freely spendable, no inscription.
```

The **pointer tag** (Ordinals tag 2) explicitly binds the inscription to the first satoshi of output 0.

## Why 546 sats?

546 sats is Bitcoin's **dust limit** — the minimum UTXO value that
Bitcoin Core nodes will relay. A 1-sat UTXO would be rejected by the
mempool as "dust." The inscription lives on exactly one sat (the first
sat of the output, per Ordinals ordinal theory), but the UTXO must
hold ≥ 546 sats to exist on the network. The other 545 sats are
padding — locked in the vault, recoverable when the UTXO is spent.

## Two Layers of Protection

1. **Economic** — dust-limit UTXO (546 sats) has no spending value to attract
2. **Consensus** — Taproot vault script prevents spending even if attempted

## Ordinals vs Runes: UTXO Sharing

A single UTXO can carry multiple ordinals (one per sat) and multiple
Rune types simultaneously. The risk profile differs:

| | Ordinals | Runes |
|---|---|---|
| **Granularity** | Individual sats | Fungible balances per UTXO |
| **Multiple per UTXO** | Yes (one per sat) | Yes (multiple Rune types) |
| **Risk of co-location** | High — sat ordering is complex, accidental transfer | Low — Runestone edicts are explicit |
| **BINST approach** | One inscription per isolated UTXO | Multiple Runes per member UTXO is fine |

Merging inscribed UTXOs is dangerous because spending scatters sats
across outputs unpredictably (ordinal numbering follows first-in-first-out
rules). The pilot isolates each inscription in its own vault UTXO to
prevent accidental loss.
