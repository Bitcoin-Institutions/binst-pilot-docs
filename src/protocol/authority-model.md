# Authority Model

> **The Bitcoin key is sovereign. Everything else is a delegate.**

## The Hierarchy

```text
Bitcoin secret key (ROOT OF AUTHORITY)
  │
  ├── controls inscription UTXO     → identity, provenance, metadata
  ├── controls Rune distribution    → membership tokens
  └── authorizes L2 contract(s)     → processing delegates
       ├── Citrea     (current)
       ├── BOB        (possible future)
       ├── Rootstock  (possible future)
       └── any L2     (portable)
```

The user who holds the Bitcoin private key has **full control** of every element in the protocol. If they decide to use a different L2, they deploy a new contract, point it at the same inscription ID, and pick up where they left off.

## What the Key Controls

| Layer | What it controls | Can the user switch it? |
|---|---|---|
| **Inscription UTXO** | Identity, metadata, provenance | No — this IS the identity |
| **Rune distribution** | Membership tokens | No — lives on Bitcoin L1 |
| **L2 contract** | Processing logic (workflows, payments) | **Yes** — redeploy to any L2 |
| **Mirror contracts** | Read-only identity/membership on other L2s *(Phase 3 — not yet built)* | **Yes** — add/remove mirrors |

## L2 Portability

Because the root of authority is the Bitcoin key (not the L2 contract address), the protocol is not locked into any specific L2. A user who starts on Citrea can later move to BOB, Rootstock, or any future Bitcoin L2 without losing their institution's identity, provenance, or membership.

Beyond simple portability, BINST supports **multi-chain presence** via a dual-channel sync model (Phase 3 plan — not yet implemented):

- **LayerZero V2** (future) — syncs identity and membership across L2s in real-time
- **Bitcoin DA** — provides trustless execution state verification via ZK batch proofs (available now)

Mirror contracts on other L2s will provide read-only identity and membership verification. Process execution stays on the home chain — **single-writer per process instance** prevents concurrent mutation conflicts across chains.

See [Cross-Chain Synchronization](./cross-chain.md) for the full model.

## Failure Modes

| Scenario | Severity | Recovery |
|---|---|---|
| L2 goes down | **Graceful** | Redeploy contracts on another L2; identity survives on Bitcoin |
| Inscription UTXO lost | **Serious** | Re-inscribe as child of original + deploy new L2 contract |
| Bitcoin key lost | **Catastrophic** | Committee 2-of-3 multi-sig recovery (Taproot vault Leaf 1) |

The degradation is intentionally hierarchical: losing the L2 is easy to recover from, losing the inscription UTXO is hard but possible, losing the Bitcoin key requires the committee backstop.

## Cryptographic Binding: `btcPubkey`

The `Institution` contract stores a `bytes32 btcPubkey` field — the
32-byte x-only public key (BIP-340) of the institution admin's Bitcoin key.

This closes the trust gap between Bitcoin identity and L2 contracts.
Without it, the link between an inscription and a contract is
informational (a stored string). With it, the binding is **verifiable**:

1. Read the institution's `btcPubkey` from the L2 contract
2. Read the inscription UTXO's owner from Bitcoin
3. Verify they match — no oracle, no trust

Citrea's Schnorr precompile (`0x0000000000000000000000000000000000000200`)
enables BIP-340 signature verification on-chain, which is the foundation
for this binding. The Rust `BitcoinIdentity` struct requires
`bitcoin_pubkey` — the L2 contract stores the same key.
