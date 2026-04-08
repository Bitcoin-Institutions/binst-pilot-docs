# BINST Pilot

> *A proof-of-concept for Bitcoin-sovereign institutional processes.*

Can complex institutional entities — institutions, process templates,
running workflows, step-by-step execution — be **permanently represented
on Bitcoin L1** and have their associated events **verified at the
Bitcoin layer**?

That is the core question. Not "decentralization" in the abstract, but
something concrete: can a Bitcoin inscription be the canonical record of
an institution's existence, and can the execution of a multi-step process
be traced back to Bitcoin cryptographically — without trusting an L2?

The pilot is a proof-of-concept for that claim. An EVM-compatible L2
(Citrea) handles the operational logic as a delegate of the Bitcoin key
holder. The L2 is replaceable; the inscription is not.

## How it works

| Concern | Approach |
|---|---|
| Institutional identity | Inscribed on Bitcoin L1 via Ordinals — the inscription IS the entity |
| Membership | Runes tokens on Bitcoin L1 — holding ≥1 token means membership |
| Operational logic | Runs on Citrea (EVM L2) as a delegate of the Bitcoin key |
| Authority | The Bitcoin key controls the inscription UTXO; the L2 contract obeys it |
| L2 replaceability | Redeploying contracts on a new L2 and binding them to the same inscription preserves identity |
| UTXO safety | Taproot script tree (NUMS + CSV + multisig) protects the inscription sat |
| Event verification | L2 batch proofs are written to Bitcoin DA — execution state is ZK-provable from Bitcoin |

If the L2 disappears, the inscription remains. If the L2 is replaced,
the same Bitcoin identity binds to the new contracts.

## What the pilot implements

- **4 smart contracts** deployed and verified on Citrea testnet —
  factory, institution, process template, process instance
- **4 Rust crates** (BINST Protocol) — decoding BINST data directly
  from Bitcoin transactions, `no_std`-compatible, WASM-ready
- **6 TypeScript scripts** — end-to-end protocol flows, Bitcoin
  inscription tooling, finality monitoring
- **`binst` metaprotocol JSON schema** — formal inscription format
  for four entity types
- **Taproot vault script tree** — NUMS + CSV + multisig UTXO
  protection for inscription sats
- **Rust/WASM webapp** — pilot user interface with real wallet
  integration (UniSat, SafePal, MetaMask), L1 inscription stack
  (PSBT batching), and L2 EVM queue (review buffer)

## What the pilot proves

1. Institutional identity can be permanently inscribed on Bitcoin L1
2. L2 contracts can operate as delegates bound to that Bitcoin identity
3. The L2 choice is non-permanent — switching L2s preserves the identity
4. Bitcoin transaction data (DA layer) can be decoded to reconstruct
   full institutional state without trusting the L2
5. Inscription UTXOs can be protected with Taproot script trees
6. A browser-native app can route L1 actions (PSBTs) and L2 actions
   (EVM calls) to the correct wallet with no mocked flows

## Source Code

| Repository | Link |
|---|---|
| Pilot | [github.com/Bitcoin-Institutions/binst-pilot](https://github.com/Bitcoin-Institutions/binst-pilot) |
| This documentation | [github.com/Bitcoin-Institutions/binst-pilot-docs](https://github.com/Bitcoin-Institutions/binst-pilot-docs) |
