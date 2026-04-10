# Discovery

How to find and verify BINST entities — from casual browsing to full trustless verification.

## Who Needs What

| What you want to know | Where to look | Full node needed? |
|---|---|---|
| Does institution X exist? | Ordinals explorer | ❌ No |
| Who is the admin? | Inscription UTXO owner | ❌ No |
| Am I a member? | Rune balance in wallet | ❌ No |
| Who are all members? | Rune indexer query | ❌ No |
| What processes exist? | Child inscriptions | ❌ No |
| What step is instance Y on? | L2 RPC (Citrea) | ❌ No |
| Is institution X anchored? | Inscription exists + L2 instances reference it | ❌ No |
| Was step execution valid? | ZK batch proof decode | ✅ Yes |
| Full trustless verification? | binst-protocol CLI | ✅ Yes |
| Which L2 is processing? | Inscription metadata | ❌ No |
| Is this person a member? (cross-chain) | Rune balance via indexer | ❌ No |
| Did step X complete? (cross-chain) | Bitcoin DA batch proof | ✅ Yes |

## Two Tiers

```text
Tier 1 (standard tooling):  Ordinals explorer + Rune wallet
  → identity, ownership, membership — no full node needed

Tier 2 (L2 RPC):  Citrea RPC + citrea-scanner --discover
  → process state, step execution, BINST contract discovery — no full node needed

Tier 3 (verification):  Bitcoin full node + binst-protocol CLI
  → ZK proof verification, state diff decoding — trustless
```

**Basic discovery requires no custom software and no full node.** Standard Ordinals and Rune tooling covers identity, ownership, membership, and provenance. Tier 2 adds on-chain contract discovery via Citrea RPC. The full node is only needed for the strongest verification tier.

### Auto-discovery with `--discover`

The `citrea-scanner` CLI can automatically crawl the factory contract to find
all BINST process instances registered on-chain:

```bash
cargo run --bin citrea-scanner -- \
  --citrea-rpc https://rpc.testnet.citrea.xyz \
  --discover \
  --factory 0x6a1d2adbac8682773ed6700d2118c709c8ce5000 \
  --block 127848
```

Discovery chain:
1. `factory.getInstanceCount()` → total number of instances
2. `factory.allInstances(i)` → instance address by index
3. `factory.getTemplateInstances(inscriptionId)` → instances for a specific L1 template
4. `factory.getUserInstances(address)` → instances created by a specific user

All discovered addresses are merged into the BINST registry, which pre-computes
storage slot hashes for matching against state diffs in ZK batch proofs.

Matched state diff values are automatically decoded into human-readable form —
addresses, integers, booleans, short Solidity strings, and packed `StepState`
structs. See [Decoding Procedure § Human-Readable Value Decoding](./decoding-procedure.md#8-human-readable-value-decoding).

Critically: **none of this depends on any specific L2 being online.**
