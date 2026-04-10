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

The user who holds the Bitcoin private key has **full control** of every element in the protocol. If they decide to use a different L2, they create new process instances on the new chain, point them at the same inscription IDs, and pick up where they left off.

## What the Key Controls

| Layer | What it controls | Can the user switch it? |
|---|---|---|
| **Inscription UTXO** | Identity, metadata, provenance | No — this IS the identity |
| **Rune distribution** | Membership tokens | No — lives on Bitcoin L1 |
| **L2 process instances** | Processing logic (workflows, payments) | **Yes** — create on any L2 |
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
| L2 goes down | **Graceful** | Create new instances on another L2; identity survives on Bitcoin |
| Inscription UTXO lost | **Serious** | Re-inscribe as child of original + create new L2 instances |
| Bitcoin key lost | **Catastrophic** | Committee 2-of-3 multi-sig recovery (Taproot vault Leaf 1) |

The degradation is intentionally hierarchical: losing the L2 is easy to recover from, losing the inscription UTXO is hard but possible, losing the Bitcoin key requires the committee backstop.

## Cryptographic Binding: `admin` Pubkey

The institution inscription body contains an `admin` field — the
32-byte x-only public key (BIP-340, hex, 64 chars) of the institution
admin's Bitcoin key. This key is the **root identity anchor** for the
entire protocol.

This closes the trust gap between Bitcoin identity and L2 execution.
Without it, the link between an inscription and an L2 process instance
is informational (a stored string). With it, the binding is **verifiable**:

1. Read the institution inscription's `admin` field from Bitcoin L1
2. Read the inscription UTXO's owner from Bitcoin
3. Verify they match — no oracle, no trust
4. L2 process instances carry a `templateInscriptionId` that chains
   back to the institution inscription, completing the link

Citrea's Schnorr precompile (`0x0000000000000000000000000000000000000200`)
enables BIP-340 signature verification on-chain, which can be used to
verify that an L2 caller is the legitimate admin. The Rust
`BitcoinIdentity` struct requires `bitcoin_pubkey` — the inscription
body stores the same key as `admin`.
