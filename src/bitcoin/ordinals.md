# Ordinals — Entity Identity

Every BINST entity is a permanent **Ordinals inscription on Bitcoin**. The inscription is the entity's birth certificate, identity anchor, and metadata carrier.

## How It Works

BINST inscriptions use the Ordinals protocol with these conventions:

- **Metaprotocol** (tag 7) = `"binst"` — filterable by any indexer
- **Content type** = `application/json`
- **Metadata** (tag 5) = CBOR-encoded structured data
- **Parent** (tag 3) = parent inscription ID (provenance chain)

## Provenance Hierarchy

Entities form a parent/child tree rooted at the institution inscription:

```text
Institution "Acme Financial" (root inscription)
 ├── Process Template "KYC Onboarding" (child)
 │    ├── Instance #1 (grandchild)
 │    │    ├── Step 1 executed by Alice (event)
 │    │    └── Step 2 executed by Bob (event)
 │    └── Instance #2
 └── Process Template "Loan Approval"
```

Anyone running `ord` can verify the full provenance chain — "KYC Onboarding was created by Acme Financial" — without touching any L2.

> **Security note — tag 3 is self-declared, not cryptographic.**
> Writing tag 3 bytes into the Ordinals envelope requires no private key;
> any party can claim any parent inscription ID.  The only cryptographically
> secure provenance proof is the parent sat being **spent as an input of the
> child's reveal transaction** — that requires the holder's private key.
> The BINST webapp distinguishes these with a three-level badge system:
> ⛓ *provenance verified* (sat-spend proven), ✓ *unverified* (tag 3 only),
> and ⬡ *declared* (JSON field only).
> See [Provenance & Parent-Child Security](./provenance.md) for the full
> security model, verification algorithm, and real-world testnet4 examples.

Each entity in the tree gets its **own sat, own UTXO, own vault**. The
parent-child relationship links them, but each inscription is
independently held and independently protected. New information about
an institution (a new process, a completed instance) is always a
**new child inscription** — never a modification of the parent.

This keeps the institution's root inscription UTXO locked and safe
while the child tree grows as the institution operates.

## What gets inscribed vs what gets ZK-proven

Not every level of the tree needs its own Ordinals inscription.
ZK batch proofs already carry all L2 state changes to Bitcoin
automatically (the sequencer pays, not the user). The practical
split:

| Level | Ordinals inscription? | Rationale |
|---|---|---|
| **Institution** | Yes — always | This IS the identity. Permanent, inscribed once. |
| **Process Template** | Yes | Proves "this process belongs to this institution" on Bitcoin. Inscribed once, never changes. |
| **Process Instance** | Optional | Created frequently. L2 state + ZK batch proof is sufficient. |
| **Step Execution** | No | High-frequency L2 operations. Already ZK-proven via batch proofs. |

The top two levels justify the inscription cost — infrequent,
high-value, permanent identity. The bottom two are operational data
better served by the L2 + ZK proof path:

```text
Institution ─────── Ordinals inscription (identity)
Process Template ── Ordinals inscription (blueprint)
Process Instance ── L2 state, verified via ZK batch proofs
Step Execution ──── L2 state, verified via ZK batch proofs
```

## Ownership

The inscription UTXO is controlled by the admin's Bitcoin key. **This key is the canonical authority** — whoever controls this UTXO controls the institution, its child entities, and any L2 contracts bound to it.

- Transfer the UTXO = transfer admin rights (on Bitcoin and all L2s)
- L2 contracts derive their authority from this key, not the other way around
- A Bitcoin maximalist holds their institution in their Bitcoin wallet

## Reinscription Policy

The first inscription is canonical (per Ordinals protocol). Reinscriptions **append** to the history — they do not overwrite:

- Inscription 1 (canonical): "Created Acme Financial, admin=pk1"
- Reinscription 2: "Updated description"
- Reinscription 3: "Admin transferred to pk2"

Institutions cannot erase their history. The append-only model matches
the transparency requirement.

**Rule: use child inscriptions for data updates. Reserve reinscription
for ownership events only** (admin transfer, key rotation). This avoids
spending the parent inscription's vault UTXO — the riskiest operation
in the protocol — for routine updates. A new process template is a new
child, not a reinscription of the institution.

## Discovery

BINST inscriptions are discoverable through standard tooling:

- **Ordinals explorers** (ordinals.com, ord.io, Hiro) — search by metaprotocol
- **Ordinals wallets** (UniSat, SafePal BTC) — shows as an asset
- **Self-hosted `ord` indexer** — trustless, complete access
- **No custom BINST software needed for basic discovery**

---

See [Mempool Pre-Consolidation](./mempool-consolidation.md) for the rules
and risk model that apply while a commit/reveal chain is in-flight between
broadcast and confirmation.
