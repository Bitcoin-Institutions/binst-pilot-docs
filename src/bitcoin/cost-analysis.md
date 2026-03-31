# Cost Analysis

Approximate costs for BINST operations at 10 sat/vB fee rate.

| Operation | Mechanism | Approx. cost |
|---|---|---|
| Create institution | Ordinal inscription (~500B text) | ~$2–5 |
| Create process template | Child inscription (~300B) | ~$1–3 |
| Record step execution | Child inscription (~200B) | ~$0.50–2 |
| Etch membership Rune | Runestone in OP_RETURN | ~$1–3 |
| Mint membership for 1 user | Runestone transaction | ~$0.50–1 |
| Transfer institution admin | Send UTXO (standard tx) | ~$0.30–1 |
| **Total: institution + template + 10 members** | | **~$15–30** |

Testnet4 is free during development. Mainnet costs scale with fee rates but remain reasonable for institutional operations that happen infrequently.

## Locked Sats at Scale

Each entity UTXO locks 546 sats (the dust limit). These sats are not
burned — they are recoverable when the vault UTXO is spent (admin
transfer, reinscription, etc.).

| Scale | Sats locked | BTC locked | At $100K/BTC |
|---|---|---|---|
| 1 institution | 546 | 0.00000546 | $0.55 |
| 100 institutions | 54,600 | 0.00054600 | $54.60 |
| 10,000 institutions | 5,460,000 | 0.05460000 | $5,460 |

Transaction fees (commit + reveal) dwarf the dust padding by 5–10×.
The real cost optimization target is batching and fee rate timing, not
the locked sats.

## Future Optimization: Batching

Bitcoin-side operations (inscribe + etch + distribute runes) can be grouped into fewer transactions:

```text
Single Bitcoin TX:
  Input:  admin's UTXO (from taproot vault)
  Output 0: inscription envelope (institution identity)
  Output 1: rune etch (INSTITUTION•MEMBER)
  Output 2: rune transfer to member_1
  Output 3: rune transfer to member_2
  Output 4: change back to vault
```

This is planned for Phase 3 — a BINST-aware transaction builder that composes Bitcoin-side operations into minimal transactions.
