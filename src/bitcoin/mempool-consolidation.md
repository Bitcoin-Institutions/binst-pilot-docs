# Mempool Pre-Consolidation

Every BINST inscription is a two-transaction chain: a **commit** transaction
and a **reveal** transaction. Between broadcast and confirmation there is a
window — the *pre-consolidation stage* — during which the inscription is live
on the network but not yet settled on-chain. This page documents the rules,
assumptions, and risks of that window, especially when parent and child
inscriptions are chained through it together.

---

## The Two-Transaction Chain

Ordinals inscriptions cannot be created in a single transaction because the
inscription script must be committed to *before* it is revealed, to prevent
fee-sniping. The pattern is:

```text
Step 1 — Commit tx
  Input:  wallet UTXO (funds)
  Output 0: P2TR UTXO whose key-path key commits to the reveal script
  Output 1: change back to wallet

Step 2 — Reveal tx
  Input 0: output 0 of the commit tx (spends the script-path)
  Input 1: parent sat UTXO (optional — required for Ordinals provenance)
  Output 0: new inscription sat (dust, 546 sats) — this is the child's identity
  Output 1: parent sat returned to change address (parent inscription survives)
  Witness:  the envelope script carrying the JSON body + tag 3
```

Each step requires one wallet signature.  
**2 signatures per inscription, minimum.**

For a batch of N inscriptions with no parent/child relationships:
`N × 2` signatures total, all independent.

For a parent + child pair (e.g. institution + process template):
`2 + 2 = 4` signatures, **sequentially dependent** — the child reveal must
spend the parent sat as input 1, which only exists after the parent reveal
is broadcast. Children must be inscribed **one at a time in sequence**; two
children cannot be signed in parallel because both would attempt to spend
the same UTXO.

---

## Signature Count Reference

| Batch contents | Minimum signatures |
|---|---|
| 1 root inscription (institution) | 2 |
| 1 institution + 1 process template | 4 |
| 2 institutions + 2 process templates | 8 |
| N institutions + M templates | (N + M) × 2 |

These counts are irreducible when Ordinals provenance (parent sat spend) is
required. The parent sat **must** be an input of the child's reveal tx, and
it only exists as a spendable UTXO after the parent reveal is broadcast.

---

## The Pre-Consolidation Window

After the parent reveal is broadcast but before it is confirmed, the network
holds its output in the **unconfirmed UTXO set**. During this window:

- Bitcoin nodes accept transactions that spend unconfirmed outputs
  (standard CPFP / child-pays-for-parent behaviour)
- The child reveal can be broadcast immediately, referencing the
  parent's unconfirmed output as input 1
- Both transactions sit in the mempool together and are typically mined
  in the same block or consecutive blocks

This is the **pre-consolidation stage**: the parent/child inscription chain
exists as a coherent unit in the mempool, not yet confirmed, but fully valid.

---

## In-Session vs Cross-Session Chaining

There are two ways a child inscription can reach the pre-consolidation stage.

### In-session (both inscriptions signed in the same stack run)

The planner assigns `ParentSource::InBatch`. After the parent reveal
succeeds, its `reveal_utxo` is passed directly to the child reveal without
any network fetch. The child is broadcast immediately after the parent.

```text
Institution reveal → broadcast → reveal_utxo captured in memory
                                       ↓
Process template reveal ← parent_utxo passed in-process
                        → broadcast (parent UTXO unconfirmed, accepted by nodes)
```

No mempool fetch, no race condition. The parent and child are always
consistent with each other. **This is the safe path.**

### Cross-session (parent was broadcast in a prior session)

The parent inscription is already in the mempool or confirmed. The user
opens a new session, adds a child to the stack, and signs. The planner
assigns `ParentSource::FetchMempool`: at sign time it calls
`fetch_inscription_sat_utxo(inscription_id)`, which queries the Xverse
Ordinals indexer for the `currentOutput` field — the **live location of
the parent sat** — then fetches that UTXO from Mempool.space.

This is the correct approach because the parent sat **moves** after every
child inscription (see [Multiple Children](#multiple-children-fan-out)
below). Fetching vout 0 of the institution's own reveal tx would fail for
any institution that already has at least one child.

**This also works**, but with one additional risk compared to in-session
chaining — see the risk table below.

---

## Multiple Children (Fan-Out)

An institution can have any number of process templates. All children
reference the **same parent inscription ID** in their tag 3 field and
`institution_id` JSON body field. The tree is a flat fan-out at the first
level of nesting:

```text
Institution  (inst_txid:0)
  ├── Template A  [tag 3 = inst_txid:0]
  ├── Template B  [tag 3 = inst_txid:0]
  └── Template C  [tag 3 = inst_txid:0]
```

Each child independently claims the same parent. The Ordinals indexer
recognises all of them as children of the institution.

### The sat travels forward through the child chain

The parent sat is spent as input 1 and returned as output 1 of every child's
reveal tx. After each child inscription it lives at a new UTXO:

```text
After institution reveal:    inst_reveal_txid : 0   ← sat starts here

After Template A inscribed:  tmpl_A_reveal_txid : 1  ← sat moved here

After Template B inscribed:  tmpl_B_reveal_txid : 1  ← sat moved here

After Template C inscribed:  tmpl_C_reveal_txid : 1  ← sat moves here
```

The institution's inscription ID (`inst_reveal_txid:0`) is **permanent and
never changes** — it is the identity of the institution. Only the UTXO
holding its sat changes as children are added.

### Sequential constraint

Because each child spends the parent sat as an input, two children
**cannot be signed in parallel** — both would reference the same UTXO and
only the first to broadcast would be accepted. Children must be inscribed
one at a time, each waiting for the previous child's reveal to be broadcast
before the next one is signed.

### How the webapp resolves the current sat location

Before signing a child's reveal, the webapp calls the Xverse Ordinals
indexer:

```
GET /v1/inscriptions/{institution_inscription_id}
→ { "currentOutput": "{txid}:{vout}", ... }
```

`currentOutput` is updated by the indexer after every confirmed block. The
webapp then calls Mempool.space to fetch the scriptpubkey and value for that
output:

```
GET /testnet4/api/tx/{txid}  →  vout[{vout}].scriptpubkey + value
```

This two-step lookup (`fetch_inscription_sat_utxo`) is correct for any
number of existing children. For in-batch parents (institution not yet
broadcast), the sat location is held in memory directly after the parent
reveal succeeds — no indexer call needed.

---

## Risk Table

| Risk | In-session | Cross-session | Mitigation |
|---|---|---|---|
| Parent UTXO not yet available | None — held in memory | Low — Mempool.space returns unconfirmed txs | `fetch_inscription_sat_utxo` falls back to vout 0 of institution reveal; clear error if fetch fails |
| Wrong UTXO fetched (sat already moved to later child) | None — in-memory reveal UTXO is always current | **Would fail** if vout 0 of institution reveal used directly | Resolved: Xverse `currentOutput` always returns the live sat location |
| Parent gets RBF-replaced before child is signed | None — both signed atomically | **Possible** — all txs are RBF-enabled (`sequence = 0xFFFFFFFD`) | Only the key-holder can initiate RBF; typical flow is uninterrupted |
| Two children signed in parallel (double-spend of parent sat) | Not possible — stack executes sequentially | Not possible — UI enforces one-at-a-time signing | Sequential execution is enforced by the async loop in `on_sign_inscribe` |
| Child references evicted parent (mempool full, low fee) | Very low — both broadcast immediately | Low on testnet4; possible on mainnet at high-fee periods | CPFP the child; both parent and child get mined together |
| `institution_id` field empty in JSON body | Not possible if `parent_ref` set correctly | Not possible if inscription ID stored in localStorage | Planner validates and warns; stack UI shows `↑ batch` / `↑ mempool` |
| Ordinals indexer sees child before parent | Indexers process by block, not mempool | Same | Non-issue — indexers only act on confirmed blocks |

---

## RBF and Our Transactions

Every transaction produced by the BINST webapp sets:

```
sequence = 0xFFFFFFFD   (Sequence::ENABLE_RBF_NO_LOCKTIME)
```

This opts every transaction into **Replace-by-Fee** signalling. The
practical effect:

- You *can* fee-bump a stuck transaction via RBF
- If you RBF the parent reveal after the child reveal is already in the
  mempool, the child reveal becomes invalid (it references an output that
  no longer exists in the mempool after the replacement)
- The child reveal would need to be rebuilt and re-signed against the
  replacement parent

**Rule:** do not RBF a parent inscription reveal if a child has already
been broadcast against it. If you need to fee-bump, use CPFP on the child
instead (add a new output spend that carries a higher fee), which pulls
both the parent and child into the next block.

---

## Assumptions Made by the Planner

The `stack_plan::build_plan` function makes these assumptions at planning time:

1. **In-batch parents always precede their children** in the stack (enforced
   by the UI — institution must be added before its templates).

2. **Cross-session parents are findable** in localStorage with a valid
   `reveal_txid`. If not present, a warning is emitted and the parent
   source falls back to an external fetch attempt.

3. **The Xverse Ordinals indexer is available** at sign time. If it is
   unreachable, `fetch_inscription_sat_utxo` falls back to vout 0 of the
   institution's own reveal tx — which is correct for a fresh institution
   with no children, but will fail for one that already has children. A
   clear error is returned and no malformed transaction is broadcast.

4. **The parent inscription was not RBF'd** between planning and signing.
   Safe in practice: only the key-holder can initiate RBF, and the typical
   flow is sequential (plan → sign → broadcast without interruption).

---

## What "Pre-Consolidation" Means for Ordinals Provenance

Ordinals provenance requires two things:

1. **Tag 3** in the reveal script envelope — the parent inscription ID
   encoded as a script push. This is present in the JSON body field
   (`institution_id`) *and* as an Ordinals envelope tag, set by the
   `txbuilder` at build time.

2. **The parent sat spent as an input** of the child's reveal tx. This is
   what the UTXO-spend proof adds: the parent sat moves from the parent
   reveal's output into the child reveal's input, and the Ordinals indexer
   traces sat ownership to confirm the relationship.

Both are satisfied during pre-consolidation:

- Tag 3 is in the witness script — it is final the moment the PSBT is signed
- The parent sat is in the mempool UTXO set and is legally spendable

The Ordinals indexer does not act on mempool state — it only processes
confirmed blocks. So from the indexer's perspective, both the parent and
child appear in confirmed blocks (possibly the same block, possibly
consecutive blocks). Provenance is established cleanly either way.

---

## Summary

The pre-consolidation stage is a normal and safe operating condition for
BINST inscriptions, subject to the following practical rules:

- **Preferred:** sign institution + templates together in one stack session
  (in-batch path, zero network dependency, zero RBF risk)
- **Acceptable:** broadcast child against an unconfirmed parent from a prior
  session (cross-session path, works because mempool accepts unconfirmed
  UTXO spends)
- **Avoid:** RBF-bumping a parent after its child has been broadcast
- **Avoid:** broadcasting a child when the parent is not yet in the mempool
  (nodes will reject it — the UTXO does not exist yet)
