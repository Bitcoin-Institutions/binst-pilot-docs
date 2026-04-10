# Provenance & Parent-Child Security

BINST uses a three-level provenance system to classify how strongly
a process template's claimed relationship to its institution has been
verified on-chain.  Understanding the security guarantees of each level
is critical: not all "on-chain" claims are equally trustworthy.

---

## The Core Question

When a process template inscription declares `"institution_id": "abc…i0"`,
how do we know the inscription was really created by the institution's admin,
and not by an arbitrary third party mimicking the relationship?

Bitcoin provides two mechanisms for this — and they have **very different
security properties**.

---

## Mechanism 1 — JSON field (`institution_id`)

The simplest declaration: the JSON body contains a field pointing to the
parent institution.

```json
{
  "type": "process_template",
  "name": "KYC Onboarding",
  "institution_id": "abc123...i0"
}
```

**Security: none.**  Anyone can write any string into a JSON body.
This is a self-declaration with no on-chain enforcement.

---

## Mechanism 2 — Ordinals tag 3 (`OP_RETURN parent`)

The Ordinals protocol defines tag 3 in the inscription envelope as a
"parent" pointer.  Indexers like Xverse and Ordiscan expose this as a
`parents` array on the inscription metadata.

```
OP_FALSE OP_IF
  ...
  03 <parent_inscription_id_bytes>   ← tag 3: parent pointer
  ...
OP_ENDIF
```

**Security: none.**  Tag 3 is arbitrary bytes written into the witness
script of the reveal transaction.  **No private key is required to write
them.**  Any inscription can claim any parent by copying 32 bytes into
the envelope.

This is often confused with cryptographic proof because it appears
"on-chain" — but the chain only records that the bytes were written,
not that the writer held any relevant key.

---

## Mechanism 3 — Parent sat spent as reveal tx input ✓

The only cryptographically secure provenance proof in Ordinals:

> The sat currently holding the parent inscription must be **spent
> as an input (`vin`) of the child's reveal transaction**.

Spending a UTXO requires a valid signature from the private key
controlling that address.  If the institution's sat appears as a vin
of the template's reveal tx, it is **mathematically proven** that the
reveal tx was authorized by whoever held the institution's private key
at that moment.

This is what the Ordinals protocol calls *parent provenance* and what
indexers use for their `/children` endpoint results.

---

## The Three-Level Badge System

The BINST webapp assigns a provenance badge to every process template card:

| Badge | Label | Mechanism | Forgeable? |
|---|---|---|---|
| `⛓` green | **provenance verified** | Parent sat spent as `vin` of reveal tx | ❌ No — requires private key |
| `✓` amber | **unverified** | Tag 3 in envelope, confirmed by indexer `parents` field | ⚠️ Yes — any witness script can contain tag 3 |
| `⬡` grey | **declared** | JSON `institution_id` field only | ⚠️ Yes — arbitrary JSON |

Only the green `⛓ provenance verified` badge constitutes a cryptographic
security proof of institutional authorship.

---

## How "verified" is determined in the webapp

The webapp uses two independent paths to establish sat-spend proof:

### Stage A — Indexer `/children` endpoint

```
GET {xverse_base}/v1/inscriptions/{institution_id}/children
```

Returns inscriptions for which the indexer has traced the institution's
sat through the reveal tx inputs — the same sat-tracking that powers
the entire Ordinals protocol.  This is authoritative.

**Limitation:** CORS-blocked from `localhost` during development.
Works correctly from any deployed domain.

### Stage B — Direct Mempool tx input check

For every template in localStorage, the webapp independently verifies
the sat-spend without relying on an indexer:

1. **Derive the reveal txid** from the inscription ID.  Inscription IDs
   always have the format `{reveal_txid}i{vout}`, so the reveal txid is
   simply the part before the `i`.

   ```
   "46f5557a10e7a75ee0b03abea6c0ecaf9f74ce7ec7d3353359744cc518ca5226i0"
    ├── reveal_txid = "46f5557a10e7a75ee0b03abea6c0ecaf9f74ce7ec7d3353359744cc518ca5226"
    └── vout        = 0
   ```

2. **Fetch the reveal tx** from the Mempool.space API — no CORS
   restriction, no API key required:

   ```
   GET https://mempool.space/testnet4/api/tx/{reveal_txid}
   ```

3. **Check the inputs** (`vin` array) against two known locations of
   the institution's sat:

   - **Genesis UTXO** — `{inst_reveal_txid}:0` — where the institution
     sat landed when the institution was first inscribed.  Matches the
     *first* child inscription (before the sat has moved).

   - **Current UTXO** — the `currentOutput` field from the Xverse
     indexer for the institution inscription.  Matches the *most recent*
     child inscription (the one that last moved the sat).

   ```
   reveal_tx.vin[i].txid == institution_genesis_txid && vin[i].vout == 0
   OR
   reveal_tx.vin[i].txid == institution_current_txid && vin[i].vout == current_vout
   ```

4. **If any vin matches** → `"verified"`.  The reveal tx was signed by
   the holder of the institution's private key.

5. **If no vin matches** → fall back to Xverse `parents` tag-3 check
   → `"linked"` (on-chain declared, not cryptographically secured).

### Coverage matrix

| Child position in chain | Stage A (prod) | Stage B genesis | Stage B current |
|---|:---:|:---:|:---:|
| First child | ✅ | ✅ | ✅ |
| Middle child | ✅ | ❌ | ❌ |
| Most recent child | ✅ | ❌ | ✅ |

Stage B covers the most common cases without requiring an indexer.
Stage A provides complete coverage in production.

---

## Why tag 3 alone is insufficient

The on-chain analysis of the BINST testnet4 pilot confirmed this
directly.  Inscriptions 62 and 63, both created by the webapp and
pointing to institution inscription 6 via tag 3, were examined:

```
Institution 6 genesis UTXO: {inst_txid}:0

Inscription 62 reveal tx (46f5…):
  vin[0]: 7bcd…:0    ← commit UTXO only; no institution sat

Inscription 63 reveal tx chain (150213… → 32ce…):
  vin[0]: 87981…:1   ← change UTXO only; no institution sat
```

Neither reveal tx spent the institution's sat as an input.  Both
inscriptions have tag 3 in their envelope pointing to institution 6,
and Xverse confirms this in the `parents` field — yet neither constitutes
cryptographic proof of authorship.

**The webapp correctly shows amber `✓ unverified` for these inscriptions,
not green `⛓ provenance verified`.**  The badge is honest: the link is
declared on-chain, but not cryptographically secured by a key-spend.

---

## Implications for the protocol

| Claim | Strength | What it proves |
|---|---|---|
| `institution_id` in JSON | Weak | Nothing — self-declared |
| Tag 3 in envelope | Weak | That the inscriber knew the parent's ID — nothing more |
| Parent sat spent as reveal vin | **Strong** | The inscriber held the private key for the institution's sat at the time of inscription |

For a production BINST deployment, only sat-spend-proven templates
should be trusted as legitimately authored by the institution admin.
The `"verified"` badge is the minimum bar for institutional trust.

---

## Relationship to Ordinals provenance rules

The sat-spend rule is not a BINST invention — it is the core mechanism
of the [Ordinals parent provenance specification](https://docs.ordinals.com/inscriptions/provenance.html):

> For a child to have a recognised parent, the parent's inscription sat
> must be an input in the child inscription's reveal transaction.

BINST's Stage B simply re-implements this check client-side against
the Mempool.space API, without needing a full Ordinals indexer.
