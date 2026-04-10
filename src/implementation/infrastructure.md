# Infrastructure & L2 Config

## Development Environment

| Component | Details |
|---|---|
| **Dev machine** (macOS) | Node.js 22+, Rust 1.94, Hardhat 3.2, `ord` 0.27 |
| **Bitcoin Core testnet4** | Remote node via SSH tunnel, `rpc:48332`, fully synced |
| **`ord` index** | Local, syncing against testnet4 node |
| **Citrea testnet** | Public RPC `https://rpc.testnet.citrea.xyz`, chain 5115 |

## Citrea Testnet Configuration

| Setting | Value |
|---|---|
| RPC | `https://rpc.testnet.citrea.xyz` |
| Chain ID | `5115` |
| EVM | Shanghai (no Cancun) |
| Currency | cBTC |
| Faucet | Citrea Discord `#faucet` |
| Explorer | [explorer.testnet.citrea.xyz](https://explorer.testnet.citrea.xyz) |

## Citrea System Contracts

| Contract | Address |
|---|---|
| Bitcoin Light Client | `0x3100000000000000000000000000000000000001` |
| Clementine Bridge | `0x3100000000000000000000000000000000000002` |
| Schnorr Precompile (BIP-340) | `0x0000000000000000000000000000000000000200` |

## BINST Deployed Contracts

| Contract | Address | Network |
|---|---|---|
| BINSTProcessFactory | `0x6a1d2adbac8682773ed6700d2118c709c8ce5000` | Citrea testnet |
| BINSTProcessFactory | `0x9fe46736679d2d9a65f0992f2272de9f3c7fa6e0` | Hardhat local |

## Clementine Bridge

- **Peg-in** (BTC → cBTC): send BTC to deposit address → bridge validates via Light Client → mints cBTC
- **Peg-out** (cBTC → BTC): burn cBTC → operator pays BTC → dispute via BitVM if needed
- **Testnet**: 100-confirmation depth, non-trivial minimum deposit. Use Discord faucet for cBTC.

## Quick Start

```bash
npm install
npx hardhat compile
npx hardhat test                        # 24 Solidity tests
cd binst-protocol && cargo test         # 80 Rust tests
cd webapp/binst-pilot-webapp && cargo test  # 97 webapp tests
npx hardhat run scripts/demo-flow.ts    # local demo
```
