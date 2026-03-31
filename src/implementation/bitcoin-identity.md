# BitcoinIdentity Type

Every BINST entity in the decoder carries a `BitcoinIdentity` struct linking it across all reachability layers.

```rust
pub struct BitcoinIdentity {
    /// Taproot x-only public key (32 bytes) — ROOT OF AUTHORITY.
    /// Controls the inscription UTXO and is the canonical identity.
    pub bitcoin_pubkey: [u8; 32],

    /// Ordinals inscription ID (e.g., "abc123...i0")
    pub inscription_id: Option<String>,

    /// Rune ID for membership token (e.g., "840000:20")
    pub membership_rune_id: Option<String>,

    /// EVM address on the current L2 (derived from or authorized by the BTC key)
    pub evm_address: Option<[u8; 20]>,

    /// HD derivation path hint (e.g., "m/86'/0'/0'/0/0")
    pub derivation_hint: Option<String>,
}
```

## Field Ordering = Authority Hierarchy

1. `bitcoin_pubkey` — the root of authority (controls inscription UTXO) — **required**
2. `inscription_id` — permanent identity on Bitcoin
3. `membership_rune_id` — membership token on Bitcoin
4. `evm_address` — current L2 delegate (**optional** — changes if L2 changes)
5. `derivation_hint` — wallet recovery aid

The required/optional distinction encodes the sovereignty model: Bitcoin identity is mandatory, L2 address is a transient delegate reference.
