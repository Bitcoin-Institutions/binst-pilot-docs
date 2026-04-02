# Test Suite

## Solidity Tests (Hardhat 3, `node:test`)

14 tests in `test/BINSTPilot.ts` covering the full contract lifecycle:

| # | Test | What it verifies |
|---|------|-----------------|
| 1 | Deploy BINSTDeployer | Factory deploys, owner set correctly |
| 2 | Create Institution | `createInstitution()` emits event, stores name/admin |
| 3 | Set Inscription ID | Admin can bind Bitcoin inscription |
| 4 | Set Rune ID | Admin can bind Rune for membership |
| 5 | Add Member | Admin adds member, `isMember` returns true |
| 6 | Remove Member | Admin removes member, `isMember` returns false |
| 7 | Create Process Template | Template created with correct step names |
| 8 | Create Process Instance | Instance linked to template and institution |
| 9 | Execute Step | First step executed, state updated |
| 10 | Execute All Steps | Sequential execution through all steps |
| 11 | Complete Instance | Final step marks instance as completed |
| 12 | Access Control | Non-admin calls revert with correct error |
| 13 | Set btcPubkey | Admin can set Bitcoin x-only pubkey |
| 14 | btcPubkey validation | Rejects zero pubkey and non-admin calls |

```bash
npx hardhat test          # runs all 14
```

## Rust Tests (`cargo test`)

68 tests across 4 crates in `taproot-reader/`:

### `binst-inscription` (10 tests)

| Test | What it verifies |
|------|-----------------|
| `extract_binst_inscription` | Parses `binst` metaprotocol envelope from witness |
| `extract_with_parent_tag` | Handles parent inscription tag |
| `no_envelope_in_random_data` | Rejects non-envelope witness data |
| `ignore_non_binst_inscription` | Skips non-`binst` metaprotocol |
| `handle_pushdata1` | Handles OP_PUSHDATA1 encoding |
| `parse_institution` | Parses institution JSON body |
| `parse_process_template` | Parses template JSON body |
| `parse_step_execution` | Parses step execution JSON body |
| `parse_state_digest` | Parses state digest body |
| `reject_unknown_type` | Rejects unknown entity type |

### `binst-decoder` (27 unit + 14 value + 5 e2e = 46 tests)

| Test | What it verifies |
|------|-----------------|
| `registry_build_lookup_populates_table` | Forward-hash lookup table is populated |
| `registry_lookup` | Registry resolves address to contract type |
| `registry_resolves_known_slot` | Known slot hash maps to BINST field |
| `map_state_diff_finds_binst_entries` | State diff entries matched to BINST |
| `map_state_diff_array_element` | Array slot elements decoded correctly |
| `decode_deployer_elements` | Deployer storage slots decoded |
| `decode_institution_simple_slots` | Institution simple fields decoded |
| `decode_institution_members_array_element` | Institution members array decoded |
| `decode_instance_completed` | Instance completion flag decoded |
| `decode_instance_step_state` | Instance step state decoded |
| `slot_to_u64_simple/large` | U256 → u64 conversion |
| `sub_words_basic/underflow` | Word subtraction arithmetic |
| `evm_storage_hash_*` | JMT key computation |
| `parse_evm_storage/header/account/index` | JMT key prefix parsing |
| `summarize_mixed_diff` | JMT diff categorization |
| `keccak256_known_vector` | Keccak256 matches known output |
| `array_base/element_slot` | Array storage layout |
| `mapping_slot_address` | Mapping storage layout |
| `add_word_offset` | U256 word offset addition |
| `full_pipeline_*` (5 e2e) | End-to-end: proof → registry → BINST changes |

#### Value decoding (`value` module — 14 tests)

| Test | What it verifies |
|------|-----------------|
| `decode_address_from_word` | Address extracted from last 20 bytes of BE word |
| `decode_uint256_small` | Small integer decoded from big-endian bytes |
| `decode_uint256_zero` | Zero value decoded correctly |
| `decode_uint256_timestamp` | Timestamp-sized integer decoded |
| `decode_bool_true` | Non-zero byte → `true` |
| `decode_bool_false` | Zero bytes → `false` |
| `decode_bytes32_pubkey` | Full 32-byte hex preserved |
| `decode_short_string` | Inline Solidity string (≤31 chars) decoded |
| `decode_short_string_empty` | Empty string slot decoded |
| `decode_long_string` | Long string length marker detected |
| `decode_step_state_completed` | Packed StepState: status + actor extracted |
| `decode_value_deleted` | `None` raw value → `DELETED` |
| `decode_value_from_hex` | Full pipeline: Citrea LE hex → BE → decoded address |
| `field_type_coverage` | Every `FieldChange` variant has a type mapping |

### `citrea-decoder` (7 tests)

| Test | What it verifies |
|------|-----------------|
| `parse_real_sequencer_commitment` | Parses real commitment from witness data |
| `reject_too_short` | Rejects truncated input |
| `batch_proof_output_roundtrip` | Proof output serialize/deserialize |
| `heuristic_finds_embedded_journal` | Heuristic journal extraction |
| `decompress_empty_fails` | Brotli rejects empty input |
| `decompress_garbage_fails` | Brotli rejects random data |
| `brotli_roundtrip` | Brotli compress → decompress roundtrip |

### `cli` (5 tests)

| Test | What it verifies |
|------|-----------------|
| `state_diff_from_rpc_basic` | RPC state diff hex map conversion |
| `state_diff_from_rpc_no_prefix` | Handles keys without `0x` prefix |
| `decode_address_array_empty` | ABI decodes empty `address[]` |
| `decode_address_array_two_addrs` | ABI decodes two-element `address[]` |
| `decode_address_array_too_short` | Rejects truncated ABI data |

```bash
cd taproot-reader && cargo test    # runs all 68
```

## Running Everything

```bash
# From project root
npx hardhat test && cd taproot-reader && cargo test
# 14 Solidity + 68 Rust = 82 total tests
```

No external services required — all tests run against local state.
