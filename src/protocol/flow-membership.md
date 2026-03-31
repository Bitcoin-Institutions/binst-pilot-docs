# Membership & Runes

## How Membership Works

Each institution etches a **Rune** on Bitcoin that represents membership. The Rune is a fungible token — holding ≥1 unit means "you are a member."

```text
Rune: ACME•MEMBER
  Divisibility: 0  (whole units only — member or not)
  Symbol: 🏛
  Premine: 1  (admin gets the first unit)
  Terms: cap=1000, amount=1 (admin mints and distributes)
```

## Operations

- **Check membership:** "Does Alice hold ≥1 `ACME•MEMBER`?" — standard Rune indexer query. No L2 needed.
- **Add member:** Admin sends 1 unit to new member's Bitcoin address.
- **Remove member:** Admin burns the token via edict, or member sends it back.
- **Visible:** Members see membership in any Rune-aware wallet (Xverse, Unisat).

## Adding a Member (Full Flow)

```text
1. Admin sends 1 INSTITUTION•MEMBER rune to new member's address
   → member now holds membership token in their Bitcoin wallet
   → visible in any Rune-aware wallet or indexer

2. Admin calls addMember(memberAddress) on the L2 contract
   → L2 contract updates member list
   → (optional: contract verifies Rune balance via bridge)

3. L2 state diff reaches Bitcoin via batch proof
   → member addition is now ZK-proven on Bitcoin
```

## L1 + L2 Mirroring

Membership exists in **two places** simultaneously:

| Layer | How membership is represented | How to check |
|---|---|---|
| **Bitcoin L1** | Rune balance (`ACME•MEMBER ≥ 1`) | Any Rune indexer |
| **L2 (Citrea)** | `isMember[address]` mapping in `Institution.sol` | EVM view call |

Both representations should be kept in sync. The Bitcoin Rune is the **authoritative** source — if there's a discrepancy, the Rune balance wins.

## Future: Governance Tokens

A separate Rune (e.g., `ACME•VOTE`) with divisibility could represent weighted voting power. Governance becomes a token distribution problem — not a staking competition.
