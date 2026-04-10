# Creating an Institution

The full lifecycle from key generation to a fully anchored Bitcoin-sovereign institution.

## Step-by-Step Flow

```text
1. Admin generates a Bitcoin key pair (x-only Taproot pubkey)
   → this key IS the institution's identity root
   → everything else derives from this key

2. Admin inscribes institution on Bitcoin (Ordinal)
   → metaprotocol: "binst", body: institution metadata
   → gets inscription ID: abc123...i0
   → inscription lives in a Taproot vault UTXO → admin owns it
   → the inscription is the institution's birth certificate

3. Admin etches membership Rune on Bitcoin
   → INSTITUTION•MEMBER, premine: 1
   → admin holds the initial unit

4. Institution is now discoverable on Bitcoin
   → any Ordinals explorer shows the inscription
   → the admin pubkey in the body identifies the controller
   → the inscription ID is the stable, cross-chain identifier

5. L2 state reaches Bitcoin via batch proof
   → institution is represented on Bitcoin via:
      a) Ordinal inscription (identity — AUTHORITATIVE)
      b) Rune (membership token)
      c) State diff in batch proof (L2 computational state)
```

> **Note:** The institution inscription ID can be referenced from any L2. If a `BINSTProcess` instance on Citrea references this inscription via `templateInscriptionId`, the provenance chain is verifiable.

> **Note:** Steps 2–3 (Bitcoin) are independent of any L2 activity. If a user creates L2 process instances before inscribing, the institution is **UNANCHORED** (see [Institution Anchoring Lifecycle](./anchoring-lifecycle.md)).

## Transaction Summary

| Step | Chain | Transaction type | Cost |
|---|---|---|---|
| Generate key | Offline | None | Free |
| Inscribe identity | Bitcoin | Ordinal inscription (~500B) | ~$2–5 |
| Etch Rune | Bitcoin | Runestone in OP_RETURN | ~$1–3 |
| Batch proof | Bitcoin | Automatic (sequencer) | User doesn't pay |

## The Inscription Envelope

Every BINST inscription uses the Ordinals envelope format:

```text
OP_FALSE OP_IF
  OP_PUSH "ord"
  OP_PUSH 1                              ← content type tag
  OP_PUSH "application/json"             ← MIME type
  OP_PUSH 7                              ← metaprotocol tag
  OP_PUSH "binst"                        ← protocol identifier
  OP_PUSH 3                              ← parent tag (optional on root)
  OP_PUSH <binst-root-inscription-id>    ← provenance chain
  OP_PUSH 0                              ← body separator
  OP_PUSH '{
    "v": 0,
    "type": "institution",
    "name": "Acme Financial",
    "admin": "a3f4b2c1d5e6f7890123456789abcdef0123456789abcdef0123456789abcdef",
    "citrea_contract": "0x1234...5678",
    "membership_rune": "ACME•MEMBER",
    "description": "Acme Financial pilot institution",
    "website": "https://acme.example"
  }'
OP_ENDIF
```

See [Inscription Schema](./inscription-schema.md) for the full JSON schema specification.
