# Post-Quantum Analysis

## The Taproot P2PK Problem

On April 2026, Bitcoin developers
[@niftynei](https://x.com/niftynei) and
[@JeremyRubin](https://x.com/JeremyRubin) highlighted a
structural property of Taproot that is relevant to every protocol built
on top of it:

> **Every P2TR output is fundamentally a Pay-to-Public-Key (P2PK).**

The 32-byte x-only public key is committed directly in the output
script.  A sufficiently powerful quantum computer could, in theory,
derive the private key from this exposed curve point before the owner
spends it.

This was a deliberate 2018/2019 design choice.  The Taproot authors
traded quantum resistance for **on-chain privacy**: all Taproot outputs
look identical regardless of the complexity of the underlying script
tree.  At the time, post-quantum (PQ) cryptography was considered too
immature and too unpredictable to constrain the design.

@niftynei summarised the rationale:

> *"Taproot accomplished its design goals to increase privacy.  Being
> quantum safe wasn't on the list of design goals.  Now we're in the era
> where PQ is a design goal."*

Jeremy Rubin added that **OP_CTV** (his covenant proposal) can be used
in a quantum-proof way, while **OP_TEMPLATEHASH** (a competing Taproot-
only proposal) cannot — precisely because OP_TEMPLATEHASH inherits the
P2PK exposure of Taproot.

## How This Affects BINST

### Attack Surface Map

| BINST Component | P2PK Exposed? | Risk | Notes |
|---|---|---|---|
| **Institution inscription UTXO** | Yes — `tb1p…` output | **Low** | NUMS internal key — no private key exists to derive |
| **ProcessTemplate inscription UTXO** | Yes — same P2TR | **Low** | Same NUMS mitigation |
| **Taproot vault (admin leaf)** | Yes — admin pubkey in script | **Medium** | Key revealed only at spend time, not before |
| **Taproot vault (committee leaf)** | Yes — 3 pubkeys in `OP_CHECKSIGADD` | **Medium** | Same: revealed only when the committee path is exercised |
| **Citrea DA inscriptions** | Yes — sequencer's Taproot output | **Not ours** | Citrea controls this key |
| **Inscription content (JSON body)** | No — pure data | **None** | Data is not a curve point |
| **State digest merkle roots** | No — SHA-256 hashes | **None** | Hash-based commitments are quantum-resistant |
| **L2 contract state** | No — EVM/ECDSA | **Separate concern** | Citrea's EVM would need its own PQ migration |
| **Bitcoin DA batch data** | No — Merkle commitments | **None** | Hash-based, quantum-resistant |

### Why BINST Is Better Positioned Than Most

**1. NUMS key kills the key-path attack.**

The primary P2PK risk is that a quantum attacker sees the output key
on-chain and derives the private key.  In BINST, the internal key is the
provably unspendable NUMS point:

```
H = lift_x(SHA256("BINST/nums"))
```

There is no discrete logarithm relationship between this point and any
known generator.  A quantum computer running Shor's algorithm needs a
*known group structure* to exploit — NUMS provides none.  The tweaked
output key `Q = H + t·G` inherits this property: knowing `Q` does not
help derive a private key because `H` has no known private key to start
with.

**2. Script-path keys are only revealed at spend time.**

The admin and committee public keys live inside the Taptree leaves, not
in the output.  They are only revealed in the witness when the
script-path is actually exercised.  An attacker would need to:

1. Wait for a vault spend transaction to appear in the mempool
2. Extract the public key from the witness
3. Run Shor's algorithm *before the transaction is mined*

With current block times (~10 minutes) and projected quantum timelines,
this is not a practical attack vector in the near term.

**3. Our covenant roadmap uses OP_CTV.**

The BINST roadmap (Phase 5) plans to use OP_CTV for covenant-locked
vaults.  Per Jeremy Rubin's analysis, OP_CTV can be constructed in a
quantum-proof way because it commits to the transaction *template hash*
rather than requiring a public key.  This means our planned upgrade path
does not inherit the P2PK vulnerability.

**4. DA proofs use hash commitments, not keys.**

The state_digest inscriptions commit to Merkle roots over instance
states.  SHA-256 Merkle trees are quantum-resistant (Grover's algorithm
provides only a quadratic speedup, requiring ~2¹²⁸ operations to break
SHA-256, which is still infeasible).  The entire DA verification chain —
from state diffs to batch proofs to digest roots — is hash-based.

## What Would Need to Change

If Bitcoin ships a post-quantum address format (e.g. a new SegWit
version using hash-based or lattice-based signatures), BINST would need
to:

### Migration Steps

1. **Move inscription UTXOs** to PQ-safe addresses.  This requires
   spending the current Taproot outputs (which reveals the script-path
   keys) and sending the sats to new PQ outputs.  The inscription
   content transfers with the sat.

2. **Update vault scripts** to use PQ signature verification opcodes
   (if available).  The MAST structure would remain; only the leaf
   scripts change from `OP_CHECKSIG` (Schnorr) to whatever PQ opcode
   Bitcoin adopts.

3. **Rotate `btcPubkey` bindings** on the L2 contracts.  The
   `setBtcPubkey()` function is currently set-once.  A PQ migration
   would require either a new setter or a contract upgrade to rebind
   to a PQ public key.

4. **Update the metaprotocol schema** to support PQ key formats in
   the `admin` field of inscription bodies (currently a 32-byte
   x-only Schnorr key).

### What We Can Do Now

| Action | Complexity | When |
|---|---|---|
| **Document the migration path** (this page) | Done | Now |
| **Avoid key reuse** — each vault uses fresh keys | Already in place | Now |
| **Don't rely on key-path spend** — NUMS is already default | Already in place | Now |
| **Design `setBtcPubkey` v2** — allow admin-initiated key rotation with timelocked delay | Low | Phase 4 |
| **Monitor BIPs** for PQ address proposals | Ongoing | — |
| **Prototype PQ inscription transfer** when a PQ address format is available on signet | Medium | When available |

## Timeline Assessment

| Milestone | Estimated | Source |
|---|---|---|
| Cryptographically relevant quantum computer | 2030–2040 | NIST, IBM roadmaps |
| Bitcoin PQ address soft fork proposal | 2026–2028 | Community discussion active |
| Bitcoin PQ soft fork activation | 2029+ | Typical activation timelines |
| BINST production deployment | 2026–2027 | Project roadmap |

The gap between BINST deployment and quantum threat is **at least 4–5
years**, and Bitcoin itself will need to solve this problem for all
Taproot users.  BINST's migration path is no harder than any other
Taproot-based protocol, and easier than most because:

- NUMS removes the key-path attack surface
- OP_CTV covenants (roadmap) are quantum-compatible
- DA verification is entirely hash-based
- Inscription UTXOs can be migrated by simple spend-and-resend

## References

- [@niftynei on Taproot P2PK design](https://x.com/niftynei/status/2039742044722041068)
- [@JeremyRubin on CTV quantum safety](https://x.com/JeremyRubin/status/2039397224954679670)
- [BIP-340: Schnorr Signatures](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)
- [BIP-341: Taproot](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki)
- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [Shor's Algorithm (Wikipedia)](https://en.wikipedia.org/wiki/Shor%27s_algorithm)
- [Grover's Algorithm (Wikipedia)](https://en.wikipedia.org/wiki/Grover%27s_algorithm)