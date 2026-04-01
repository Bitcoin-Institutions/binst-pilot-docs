# Summary

[Introduction](./introduction.md)

---

# Protocol

- [Architecture](./protocol/architecture.md)
    - [The Three-Layer Model](./protocol/three-layers.md)
    - [Authority Model](./protocol/authority-model.md)
    - [Institution Anchoring Lifecycle](./protocol/anchoring-lifecycle.md)
- [Read/Write Phase Model](./protocol/read-write-phases.md)
- [Creating an Institution](./protocol/flow-create-institution.md)
- [Membership & Runes](./protocol/flow-membership.md)
- [Process Execution](./protocol/flow-process-execution.md)
- [Switching L2s](./protocol/flow-switching-l2s.md)
- [Admin Transfer](./protocol/flow-admin-transfer.md)
- [Cross-Chain Synchronization](./protocol/cross-chain.md)
- [Inscription Schema](./protocol/inscription-schema.md)

# Bitcoin Integration

- [Ordinals — Entity Identity](./bitcoin/ordinals.md)
- [Runes — Membership Tokens](./bitcoin/runes.md)
- [Taproot Vault — UTXO Safety](./bitcoin/taproot-vault.md)
    - [Vault Unlock Flows](./bitcoin/vault-unlock.md)
    - [Sat Isolation](./bitcoin/sat-isolation.md)
    - [Graceful Degradation](./bitcoin/degradation.md)
    - [Taproot Coverage Audit](./bitcoin/taproot-coverage.md)
- [ZK Batch Proofs — Computational Integrity](./bitcoin/batch-proofs.md)
    - [Citrea DA Transaction Format](./bitcoin/citrea-da-format.md)
    - [Decoding Procedure](./bitcoin/decoding-procedure.md)
- [Discovery](./bitcoin/discovery.md)
- [Cost Analysis](./bitcoin/cost-analysis.md)

# Implementation

- [Smart Contracts](./implementation/contracts.md)
- [Taproot Reader (Rust)](./implementation/taproot-reader.md)
    - [BitcoinIdentity Type](./implementation/bitcoin-identity.md)
- [Scripts & Tooling](./implementation/scripts.md)
- [Infrastructure & L2 Config](./implementation/infrastructure.md)
- [Test Suite](./implementation/tests.md)

---

[Use Cases](./use-cases.md)
[Contributing](./contributing.md)
[References](./references.md)
