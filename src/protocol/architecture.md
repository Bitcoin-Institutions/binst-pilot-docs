# Architecture

The architecture is built on a **three-layer sovereignty model** following a
**"Thin L2, Fat L1"** philosophy: Bitcoin is the authoritative identity and
definition layer; the L2 is a minimal execution delegate.

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
│  BINSTProcessFactory ─── Thin factory (1 per chain)         │
│  BINSTProcess ────────── Self-contained instance with       │
│                          embedded steps + L1 inscription    │
│                          anchor (templateInscriptionId)     │
│                                                             │
│  No institution or template contracts on L2 — those live    │
│  on Bitcoin as inscriptions. L2 only holds execution state. │
│                                                             │
│  Execution state verified trustlessly via Bitcoin DA proofs  │
└──────────────────────────┬──────────────────────────────────┘
                           │ verified by
┌──────────────────────────▼──────────────────────────────────┐
│         VERIFICATION LAYER                                   │
│                                                             │
│  BitVM2 ─────── Trust-minimized bridge verification         │
│                 (operational on Citrea via Clementine)       │
│  ZK Proofs ──── Batch proofs of L2 state transitions        │
│                 inscribed on Bitcoin (Groth16/RISC Zero)     │
│  Covenants ──── Future: native BTC spending constraints     │
│  SNARK ──────── Future: ZK proof verification in Script     │
└─────────────────────────────────────────────────────────────┘
```

Each layer serves a distinct purpose:

- **Bitcoin L1** — permanent identity (inscriptions), membership (Runes), and the root of authority
- **L2 Processing** — minimal execution delegate; only holds step-by-step process state, not identity
- **Verification** — trust-minimized verification between L2 and Bitcoin (BitVM2 operational on Citrea; ZK batch proofs inscribed on Bitcoin)

> **Thin L2 principle:** The L2 has no institution or template contracts.
> Those concepts live entirely on Bitcoin as inscriptions. Each
> `BINSTProcess` instance carries a `templateInscriptionId` that anchors
> it back to Bitcoin L1 — the L2 is a pure execution engine.

> **Pilot scope note:** Cross-chain identity mirroring via LayerZero V2 is an architectural plan (Phase 3) — not implemented in the current pilot. The pilot runs on a single L2 (Citrea). See [Cross-Chain Synchronization](./cross-chain.md) for the design.
