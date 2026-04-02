# Scripts & Tooling

Six TypeScript scripts demonstrate the protocol end-to-end.

| Script | Purpose |
|---|---|
| `demo-flow.ts` | End-to-end: deploy → institution → members → process → execute all steps |
| `inscribe-binst.ts` | Generate `ord` commands to inscribe BINST entities on Bitcoin testnet4. Requires `BTC_RPC_PASS` in `.env` |
| `taproot-vault.ts` | Build Taproot leaf scripts for inscription UTXO safety (NUMS + CSV + multisig) |
| `bitcoin-awareness.ts` | Read Bitcoin Light Client, query finality RPCs |
| `finality-monitor.ts` | Poll Citrea RPCs until a watched L2 block is committed / ZK-proven |
| `test-protocol.ts` | Query live deployed contracts on Citrea testnet |

## Usage Examples

```bash
# Run end-to-end demo on local Hardhat
npx hardhat run scripts/demo-flow.ts

# Deploy to Citrea Testnet
npx hardhat run scripts/demo-flow.ts --network citreaTestnet

# Generate inscription command
npx ts-node scripts/inscribe-binst.ts institution "Acme Financial" <admin_pubkey>

# Generate Taproot vault scripts
npx ts-node scripts/taproot-vault.ts <admin_pk> <committee_A> <committee_B> <committee_C>

# Bitcoin awareness (reads Light Client)
npx tsx scripts/bitcoin-awareness.ts

# Monitor finality for a specific L2 block
WATCH_L2=23972426 npx tsx scripts/finality-monitor.ts
```
