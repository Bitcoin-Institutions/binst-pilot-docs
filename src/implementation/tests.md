# Test Suite

## Solidity Tests (Hardhat 3, `node:test`)

12 tests in `test/BINSTPilot.ts` covering the full contract lifecycle:

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

```bash
npx hardhat test          # runs all 12
```

## Rust Tests (`cargo test`)

16 tests across 4 crates in `taproot-reader/`:

### `binst-inscription` (9 tests)

| Test | What it verifies |
|------|-----------------|
| `parse_institution_inscription` | Parses institution JSON from Ordinals envelope |
| `parse_process_template` | Parses template inscription |
| `parse_process_instance` | Parses instance inscription |
| `parse_step_execution` | Parses step execution inscription |
| `reject_wrong_metaprotocol` | Rejects non-`binst` metaprotocol tag |
| `reject_invalid_json` | Rejects malformed JSON body |
| `validate_schema_institution` | Validates against JSON Schema |
| `validate_schema_template` | Validates template against schema |
| `roundtrip_serialize` | Serialize → deserialize preserves all fields |

### `binst-decoder` (5 tests)

| Test | What it verifies |
|------|-----------------|
| `decode_institution_storage` | Maps L2 storage slots → `InstitutionState` |
| `decode_process_template_storage` | Maps slots → `ProcessTemplateState` |
| `decode_process_instance_storage` | Maps slots → `ProcessInstanceState` |
| `build_bitcoin_identity` | Constructs `BitcoinIdentity` from decoded data |
| `correlate_inscription_to_state` | Links inscription ID to L2 entity |

### `citrea-decoder` (2 tests)

| Test | What it verifies |
|------|-----------------|
| `parse_complete_da_blob` | Parses full `DataOnDa` Borsh-encoded blob |
| `parse_sequencer_commitment` | Extracts commitment from witness data |

```bash
cd taproot-reader && cargo test    # runs all 16
```

## Running Everything

```bash
# From project root
npx hardhat test && cd taproot-reader && cargo test
```

No external services required — all tests run against local state.
