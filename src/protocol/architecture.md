# Architecture

The architecture is built on a **three-layer sovereignty model**.

```text
┌─────────────────────────────────────────────────────────────┐
│              BITCOIN L1 (Authority)                          │
│                                                             │
│  Ordinals Inscriptions ──── Institutional Identity          │
│  Runes Tokens ───────────── Membership & Roles              │
│  Tapscript Vault ────────── UTXO Safety Layer               │
│  BTC Key ────────────────── Root of All Authority           │
└──────────────────────────┬──────────────────────────────────┘
                           │ anchors
┌──────────────────────────▼──────────────────────────────────┐
│           L2 PROCESSING LAYER (Delegate)                     │
│           Currently: Citrea (Chain 5115)                     │
│                                                             │
│  Institution.sol ─────── On-chain institution definition    │
│  ProcessTemplate.sol ─── Immutable workflow logic           │
│  ProcessInstance.sol ─── Running execution + state          │
│  BINSTDeployer.sol ───── Factory & registry                 │
│                                                             │
│  Cross-chain: LayerZero V2 mirrors identity to other L2s    │
│  Execution state verified trustlessly via Bitcoin DA proofs  │
└──────────────────────────┬──────────────────────────────────┘
                           │ future
┌──────────────────────────▼──────────────────────────────────┐
│         VERIFICATION LAYER (Future)                          │
│                                                             │
│  BitVM ──────── Fraud-proof verification on BTC             │
│  BitVMX ─────── RISC-V execution verification               │
│  Covenants ──── Native BTC spending constraints             │
│  SNARK ──────── ZK proof verification on BTC                │
└─────────────────────────────────────────────────────────────┘
```

Each layer serves a distinct purpose:

- **Bitcoin L1** — permanent identity, membership, and the root of authority
- **L2 Processing** — complex logic execution as a delegate of the Bitcoin key holder
- **Verification** (future) — trust-minimized verification between L2 and Bitcoin
