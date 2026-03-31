# Runes — Membership Tokens

Each institution etches a **Rune** that represents membership on Bitcoin L1.

## Configuration

```text
Rune: ACME•MEMBER
  Divisibility: 0  (whole units only — member or not)
  Symbol: 🏛
  Premine: 1  (admin gets the first unit)
  Terms: cap=1000, amount=1 (admin mints and distributes)
```

## Operations

| Action | How | Who |
|---|---|---|
| Check membership | Query Rune balance ≥ 1 | Anyone |
| Add member | Send 1 unit to member's Bitcoin address | Admin |
| Remove member | Burn via edict, or member sends back | Admin or member |
| View membership | Any Rune-aware wallet | Member |

## No Custom Software Needed

Membership is a standard Rune balance check — any Rune indexer, any Rune-aware wallet (Xverse, Unisat), or a self-hosted `ord` can verify it. No BINST-specific tooling required for basic membership queries.

## Future: Governance Tokens

A separate Rune (e.g., `ACME•VOTE`) with divisibility could represent weighted voting power. Governance becomes a token distribution problem — not a staking competition.
