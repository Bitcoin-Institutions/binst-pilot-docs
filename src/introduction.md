# BINST Pilot

> *A proof-of-concept for Bitcoin-sovereign institutional processes.*

This pilot implements a three-layer architecture for institutional
operations where **the Bitcoin key is the root of authority**, Ordinals
inscriptions carry institutional identity, and an EVM-compatible L2
executes operational logic as a portable delegate.

## The problem

Current approaches to on-chain institutional operations either:

- Run everything on an L2, with no Bitcoin anchor — identity is L2-dependent
- Use Bitcoin only for settlement, with no institutional semantics
- Lock users into a single L2 with no portability

## The pilot's approach

| Concern | How the pilot handles it |
|---|---|
| Institutional identity | Inscribed on Bitcoin L1 via Ordinals — permanent, sovereign |
| Membership | Represented by Runes tokens on Bitcoin L1 |
| Operational logic | Runs on an L2 (Citrea) as a portable delegate |
| Authority | The Bitcoin key holder controls everything; the L2 contract obeys |
| L2 portability | Switching L2s means redeploying contracts bound to the same inscription |
| UTXO safety | Taproot script tree (NUMS + CSV + multisig) protects inscription sats |
| Bitcoin verification | Taproot Reader decodes L2 state directly from Bitcoin DA transactions |

If the L2 disappears, the inscription remains. If the L2 is replaced,
the same Bitcoin identity binds to the new contracts.

## What the pilot implements

- **4 smart contracts** deployed and verified on Citrea testnet —
  factory, institution, process template, process instance
- **4 Rust crates** (Taproot Reader) — decoding BINST data directly
  from Bitcoin transactions, `no_std`-compatible, WASM-ready
- **6 TypeScript scripts** — end-to-end protocol flows, Bitcoin
  inscription tooling, finality monitoring
- **`binst` metaprotocol JSON schema** — formal inscription format
  for four entity types
- **Taproot vault script tree** — NUMS + CSV + multisig UTXO
  protection for inscription sats

## What the pilot proves

1. Institutional identity can be permanently inscribed on Bitcoin L1
2. L2 contracts can operate as delegates bound to that Bitcoin identity
3. The L2 choice is non-permanent — switching L2s preserves the identity
4. Bitcoin transaction data (DA layer) can be decoded to reconstruct
   full institutional state without trusting the L2
5. Inscription UTXOs can be protected with Taproot script trees

## Source Code

| Repository | Link |
|---|---|
| Pilot | [github.com/Bitcoin-Institutions/binst-pilot](https://github.com/Bitcoin-Institutions/binst-pilot) |
| This documentation | [github.com/Bitcoin-Institutions/binst-pilot-docs](https://github.com/Bitcoin-Institutions/binst-pilot-docs) |
