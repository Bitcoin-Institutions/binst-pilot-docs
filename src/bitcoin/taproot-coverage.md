# Taproot Coverage Audit

BINST was designed to live as close to Bitcoin L1 as possible. This page cross-references every feature introduced by the Taproot upgrade (BIPs 340, 341, 342) against what the pilot actually uses, deliberately skips, or defers to production.

## Features We Use

| Taproot Feature | BIP | Where We Use It |
|---|---|---|
| **P2TR output** (SegWit v1) | 341 | Vault address (`tb1p...` / `bc1p...`) — every inscription UTXO is locked to a Pay-to-Taproot output |
| **x-only public keys** (32 bytes) | 340 | Admin key, committee keys, NUMS internal key — all stored and transmitted as 32-byte x-only keys, saving 1 byte vs compressed ECDSA |
| **Schnorr signatures** (64 bytes) | 340 | Admin single-sig (Leaf 0) and committee multi-sig (Leaf 1) — 64-byte Schnorr sigs replace 71–72-byte ECDSA sigs |
| **MAST (Merkelized Alternative Script Tree)** | 341 | 2-leaf script tree: Leaf 0 (admin + CSV) and Leaf 1 (committee override). Only the exercised leaf is revealed on-chain — the other stays hidden |
| **Tagged hashes** | 341 | `TapLeaf`, `TapBranch`, `TapTweak` — domain-separated SHA-256 used for tree construction and output key derivation |
| **Taptweak** (Q = P + t·G) | 341 | NUMS internal key + Merkle root → tweaked output key. This is the core of BIP 341 key commitment |
| **NUMS internal key** | 341 | Provably unspendable point — disables key-path spend entirely, forcing all spends through the script tree |
| **Script-path spend** | 341 | Both vault unlock paths (admin and committee) use Taproot script-path spending with control blocks |
| **Control blocks** | 341 | `(parity \| leaf_version) \|\| internal_key \|\| sibling_hash` — built for both leaves, enabling Taproot proof-of-inclusion |
| **Leaf version 0xc0** | 342 | All leaf scripts use the BIP 342 Tapscript leaf version |
| **OP_CHECKSIG (Schnorr variant)** | 342 | Admin leaf — single-key Schnorr signature check |
| **OP_CHECKSIGADD** | 342 | Committee leaf — the BIP 342 replacement for `OP_CHECKMULTISIG` (which is disabled in Tapscript). Accumulates a counter across multiple signature checks |
| **OP_CHECKSEQUENCEVERIFY (CSV)** | 112 | Admin leaf — enforces 144-block (~24h) relative timelock before the admin can move the inscription UTXO |
| **Schnorr precompile on Citrea** | 340 | Citrea's precompile at `0x…0200` can verify BIP-340 Schnorr signatures in Solidity — used for Bitcoin-key-based L2 authorization |
| **Ordinals inscription in witness** | 341 | The inscription envelope lives inside Tapscript witness data. Taproot's witness discount makes inscriptions economically viable |
| **Bech32m address encoding** | 341 | All P2TR addresses use Bech32m (BIP 350), distinct from SegWit v0's Bech32 |

## Features We Deliberately Skip

| Taproot Feature | BIP | Why We Skip It |
|---|---|---|
| **Key-path spend** | 341 | We kill it with the NUMS internal key. The entire purpose of the vault is to prevent accidental spending — a live key-path would let any Taproot-aware wallet move the inscription UTXO. |
| **Key aggregation / MuSig2** | 340 | MuSig2 aggregates N public keys into a single key for key-path spend. Since we disable the key-path, there is no aggregated key to spend with. For the committee, we use `OP_CHECKSIGADD` in a script leaf instead — this is simpler, independently auditable, and requires no interactive MuSig signing rounds between committee members. |
| **Batch signature verification** | 340 | Batch verification is a **node-level optimization**: Bitcoin Core can verify N Schnorr signatures faster than verifying them individually. This is transparent to script authors — our transactions benefit automatically when nodes use batch validation. There is nothing to implement. |
| **Annex field** | 341 | The annex is a reserved witness field (identified by the `0x50` prefix) with no current consensus meaning. It is reserved for future soft-fork extensions. No use case today. |
| **OP_SUCCESS opcodes** | 342 | These are upgrade placeholders that make unrecognized opcodes succeed unconditionally, allowing future soft forks to assign them new semantics. They exist precisely so that new opcodes can be added without a hard fork. Nothing to use today. |

## Design Rationale

### Why kill the key-path?

In a normal P2TR workflow, the key-path is the happy path — it looks like a regular single-sig spend and provides maximum privacy. BINST deliberately sacrifices this because the vault's primary job is to **prevent** spending:

- The inscription UTXO is the root of authority
- Accidental spending = loss of the institution
- The NUMS point makes the key-path provably dead
- All spends must go through a script leaf with explicit conditions

This is the correct tradeoff for a protocol where the UTXO represents identity, not money.

### Why OP_CHECKSIGADD instead of MuSig2?

MuSig2 would produce a cleaner on-chain footprint (single key, single sig), but it requires:

1. **Interactive signing rounds** between committee members
2. **Nonce commitment coordination** (two rounds minimum)
3. **Specialized MuSig2 software** on each signer's machine
4. All parties online at roughly the same time

`OP_CHECKSIGADD` in a Tapscript leaf is:

1. **Non-interactive** — each member signs independently
2. **Standard tooling** — any BIP-340 Schnorr signer works
3. **Auditable** — the script is human-readable and each signature is individually verifiable
4. **PSBT-compatible** — members sign via PSBT (BIP 174/371) without being online simultaneously

For an emergency recovery mechanism (which the committee path is), simplicity and independence matter more than on-chain size.

## Potential Future Enhancements

These Taproot-adjacent features are not in the pilot but could be adopted in production:

| Enhancement | Basis | What It Enables | Complexity |
|---|---|---|---|
| **MuSig2 aggregated key-path** (separate vault variant) | BIP 340 | Aggregate committee keys into one key for key-path spend. Looks like a regular P2TR on-chain → maximum privacy. Script tree remains as fallback. | High — requires MuSig2 signing infrastructure |
| **Deeper MAST trees** (3+ leaves) | BIP 341 | Add a third leaf: e.g., a dead-man switch that becomes spendable after 1 year of inactivity, or a dedicated "migrate to covenant vault" leaf | Medium — straightforward script extension |
| **Schnorr-signed L2 actions** (single-wallet UX) | BIP 340 | Admin signs L2 transactions with their Bitcoin Schnorr key via Citrea's precompile → one wallet, one identity, both layers | Medium — needs account abstraction on L2 |
| **Covenants** (OP_CTV / OP_CAT) | Proposed | Restrict the output of a vault spend — the UTXO can only move to a pre-approved address. True on-chain spending constraints. | Requires soft fork (not yet activated) |
| **Batch inscriptions in one tree** | BIP 341 | Embed multiple inscription commitments in different MAST leaves of a single transaction, reducing on-chain cost for multi-template institutions | Low–Medium |

## Summary

BINST uses **every active Taproot feature** relevant to its design goal of UTXO-locked institutional identity:

- ✅ P2TR outputs with Bech32m
- ✅ MAST script tree (2 leaves, hidden until spent)
- ✅ Schnorr signatures (64 bytes, provably secure)
- ✅ OP_CHECKSIGADD (BIP 342 multisig)
- ✅ Tagged hashes and taptweak for key commitment
- ✅ NUMS point to disable key-path
- ✅ Control blocks for script-path proofs
- ✅ CSV timelocks in Tapscript
- ✅ Schnorr precompile on L2

The features we skip (key-path, MuSig2, annex, OP_SUCCESS) are either deliberately disabled for security, irrelevant to script authors, or reserved for future upgrades. No Taproot capability is left on the table without a documented reason.

### References

- [BIP 340 — Schnorr Signatures](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)
- [BIP 341 — Taproot](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki)
- [BIP 342 — Tapscript](https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki)
- [River — What Is Taproot?](https://river.com/learn/what-is-taproot/)
- [River — BIP 341](https://river.com/learn/terms/b/bip-341-taproot/)
- [River — BIP 342](https://river.com/learn/terms/b/bip-342-tapscript/)
- [River — BIP 340](https://river.com/learn/terms/b/bip-340-schnorr-signatures/)
