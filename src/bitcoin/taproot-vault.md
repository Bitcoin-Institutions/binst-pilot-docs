# Taproot Vault — UTXO Safety

The inscription UTXO is the root of authority. Losing it means losing the institution. The Taproot vault prevents accidental spending at the **consensus level**.

## The Risk

An admin who spends the inscription UTXO in a non-Ordinals-aware wallet loses control of the inscription. The data remains on Bitcoin permanently, but the UTXO tracking it moves to an unknown party.

**This is the most critical risk in the protocol.** The vault is not optional — it is essential infrastructure.

## Miniscript Policy

The vault is defined as a [BIP 379](https://github.com/bitcoin/bips/blob/master/bip-0379.md) miniscript policy:

```text
or(
  and(pk(admin), older(144)),       ← admin transfer, ~24h CSV delay
  thresh(2, pk(A), pk(B), pk(C))    ← 2-of-3 committee, immediate
)
```

This policy is compiled to a Taproot descriptor using `rust-miniscript`:

```text
tr(NUMS, { and(pk(admin),older(144)), thresh(2,pk(A),pk(B),pk(C)) })
```

- **Internal key** = NUMS point (unspendable — disables key-path spend)
- **Leaf 0** = admin single-sig + 144-block CSV delay
- **Leaf 1** = 2-of-3 committee multisig (immediate)

### Why Miniscript?

The previous implementation used hand-rolled Tapscript (`taproot-vault.ts`). Miniscript gives us:

1. **Wallet compatibility** — the descriptor is importable into Sparrow, Liana, Nunchuk, and any BIP 379 wallet. Users can sign vault spends with standard software.
2. **Compiler-verified correctness** — the miniscript compiler guarantees the spending conditions match the policy. No hand-rolled opcode bugs.
3. **Witness size analysis** — the compiler provides worst-case witness sizes for fee estimation.
4. **Extensibility** — new policies (timelocked multisig, decay trees) are a one-line policy change.

## Script Structure

The compiled descriptor produces the same Taproot structure:

```text
Taproot output:
  Internal key: NUMS point (unspendable — disables key-path spend)

  Script tree:
    Leaf 0 (admin transfer — time-delayed):
      <admin_pubkey> OP_CHECKSIG
      <144> OP_CHECKSEQUENCEVERIFY OP_DROP     ← ~24h delay

    Leaf 1 (committee override — immediate):
      <key_A> OP_CHECKSIG
      <key_B> OP_CHECKSIGADD
      <key_C> OP_CHECKSIGADD
      <2> OP_NUMEQUAL
```

## How It Works

| Path | Who | Delay | Purpose |
|---|---|---|---|
| Key path | Nobody | ∞ | Disabled (NUMS internal key) — no accidental spend possible |
| Leaf 0 | Admin (single key) | ~24 hours (144 blocks CSV) | Deliberate admin transfer with safety delay |
| Leaf 1 | 2-of-3 committee | Immediate | Emergency override for key loss or compromise |

## Rust Implementation

The vault module lives in `binst-decoder/src/vault.rs`:

```rust
use binst_decoder::vault::{VaultPolicy, parse_xonly};

let policy = VaultPolicy::new(
    parse_xonly("79be667e…").unwrap(),
    [parse_xonly("c6047f94…").unwrap(),
     parse_xonly("f9308a01…").unwrap(),
     parse_xonly("e493dbf1…").unwrap()],
);

let desc = policy.compile().unwrap();
println!("{}", desc.descriptor);        // tr(NUMS, {…})
println!("{}", desc.address_testnet);   // tb1p…
println!("{}", desc.address_mainnet);   // bc1p…

for path in &desc.spending_paths {
    println!("{}: {} keys, {:?} CSV, ~{} vbytes",
        path.name, path.required_keys.len(),
        path.timelock_blocks, path.witness_size);
}
```

The module also compiles to WASM (`--features wasm`) for in-browser vault generation.

## Why This Works

- **No accidental spending** — the key path is dead. Regular wallets can't spend it.
- **Admin retains control** — Leaf 0 allows deliberate moves with a CSV safety net.
- **Committee backstop** — Leaf 1 is the "break glass" emergency path.
- **Standard Bitcoin** — uses only Taproot (BIP 341/342), OP_CHECKSIG, OP_CSV, OP_CHECKSIGADD. No soft fork needed.
- **Wallet-native** — the descriptor imports directly into BIP 379 wallets (Sparrow, Liana, Nunchuk).
- **Ordinals-compatible** — `ord` tracks inscriptions regardless of spending script.

**Future enhancement:** When OP_CTV or OP_CAT activates, the vault can enforce that the UTXO is only spendable to pre-approved addresses (true covenant protection).

See also: [Vault Unlock Flows](./vault-unlock.md) · [Sat Isolation](./sat-isolation.md) · [Graceful Degradation](./degradation.md)
