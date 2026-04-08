# Scripts & Tooling

Six TypeScript scripts demonstrate the protocol end-to-end from the CLI.
The [WASM Webapp](./webapp.md) provides the browser-native equivalent
for institution creation, process design, and step execution.

| Script | Purpose | Status |
|---|---|---|
| `demo-flow.ts` | End-to-end: deploy → institution → members → process → execute all steps | Active |
| `inscribe-binst.ts` | Generate `ord` commands to inscribe BINST entities on Bitcoin testnet4 | Active |
| `taproot-vault.ts` | Build Taproot leaf scripts for inscription UTXO safety (NUMS + CSV + multisig) | **Deprecated** — replaced by `binst-decoder::vault` (Rust miniscript) |
| `psbt-transfer.ts` | Generate PSBT commands for atomic vault transfers | **Deprecated** — replaced by wallet-native descriptor signing |
| `bitcoin-awareness.ts` | Read Bitcoin Light Client, query finality RPCs | Active |
| `finality-monitor.ts` | Poll Citrea RPCs until a watched L2 block is committed / ZK-proven | Active |
| `test-protocol.ts` | Query live deployed contracts on Citrea testnet | Active |

> **Note:** `taproot-vault.ts` and `psbt-transfer.ts` are kept as reference implementations. The Rust vault module (`binst-decoder/src/vault.rs`) uses BIP 379 miniscript to produce wallet-compatible Taproot descriptors, replacing the hand-rolled scripts. See [Taproot Vault](../bitcoin/taproot-vault.md).

## Usage Examples

```bash
# Run end-to-end demo on local Hardhat
npx hardhat run scripts/demo-flow.ts

# Deploy to Citrea Testnet
npx hardhat run scripts/demo-flow.ts --network citreaTestnet

# Generate inscription command
npx ts-node scripts/inscribe-binst.ts institution "Acme Financial" <admin_pubkey>

# Generate vault descriptor (Rust — replaces taproot-vault.ts)
cd binst-protocol && cargo test -p binst-decoder vault

# Bitcoin awareness (reads Light Client)
npx tsx scripts/bitcoin-awareness.ts

# Monitor finality for a specific L2 block
WATCH_L2=23972426 npx tsx scripts/finality-monitor.ts
```
