# Read/Write Phase Model

## Two Transaction Domains

BINST operations happen in two independent domains:

1. **Bitcoin transactions** — deliberate, user-initiated actions that create or transfer identity (inscriptions, runes)
2. **L2 transactions** — EVM transactions on the processing delegate for institutional logic

These domains are **decoupled**. A Bitcoin inscription and an L2 process instance are separate operations that can happen in any order. The batch proof that anchors L2 state to Bitcoin happens **automatically** — the user doesn't trigger it.

```text
USER ACTION           L2 (Citrea)                    Bitcoin
─────────────────────────────────────────────────────────────
Inscribe Institution  (nothing on L2)                ← ordinal inscription
                                                        (user-initiated)

Inscribe Template     (nothing on L2)                ← child inscription
                                                        (linked to institution)

Create Instance       → factory.createInstance()      (nothing yet)
                      → BINSTProcess deployed on L2

                      ... L2 batches state ...     
                      
                      → batch proof inscribed          ← Bitcoin DA write
                                                         (automatic, not 
                                                          user-initiated)

Execute Steps         → instance.executeStep()        (nothing yet)
                      → step state updated on L2

                      ... L2 batches state ...

                      → batch proof inscribed          ← Bitcoin DA write
```

## Write Phases (User-Initiated Transactions)

| Action | Where | Who pays / signs |
|---|---|---|
| Inscribe institution | Bitcoin (ordinal) | User, BTC wallet |
| Inscribe process template | Bitcoin (ordinal, child) | User, BTC wallet |
| Etch membership Rune | Bitcoin (rune) | User, BTC wallet |
| Send Rune to member | Bitcoin (rune) | Admin, BTC wallet |
| Create process instance | Citrea (EVM) | Admin, EVM wallet |
| Execute step | Citrea (EVM) | Authorized user, EVM wallet |

## Automatic (No User Action)

| Action | Where | Who pays |
|---|---|---|
| ZK batch proof | Bitcoin DA | Citrea sequencer (periodic, async) |

The user does **not** create a Bitcoin transaction when they interact with Citrea. The batch proof is automatic — the sequencer batches L2 state changes and inscribes the ZK proof on Bitcoin periodically. The user doesn't trigger it or pay for it.

## Read Phases (Free)

| Action | Where | Cost |
|---|---|---|
| Verify membership | Citrea (EVM view call) | Free |
| Check process state | Citrea (EVM view call) | Free |
| Verify inscription exists | Bitcoin (indexer query) | Free |
| Verify batch proof | Bitcoin DA (decode) | Free |

## Wallet UX

### Current: Two Wallets

- **Bitcoin wallet** (UniSat, SafePal BTC) — for inscriptions and runes
- **EVM wallet** (MetaMask, SafePal EVM) — for L2 transactions

The Schnorr precompile on Citrea (`0x5a`) means contracts *can verify* Bitcoin Schnorr signatures, but the current flow requires both wallets.

### Future: Single Bitcoin Wallet

Account abstraction or Schnorr-verified sessions will allow the user to sign once with their Bitcoin key, and an AA layer submits to the L2. One wallet, one identity.
