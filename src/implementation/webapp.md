# WASM Webapp

A Rust/WASM single-page application that implements the BINST user-facing
pilot flows — institution creation, process design, and step execution.
Built with [Trunk](https://trunkrs.dev/), targeting `wasm32-unknown-unknown`.

Network: **Bitcoin Testnet4** and **Citrea testnet**.

## Purpose

The webapp is the pilot's **honest frontend**: every write operation that
touches Bitcoin or Citrea goes through a real wallet. There are no mocked
broadcasts, no simulated signing prompts. If an action requires a
transaction, the wallet pops up.

---

## Signing model — what requires a signature and where

Two entirely different signing mechanisms are used, one per layer:

| Layer | Technology | What is signed | Wallets |
|-------|-----------|----------------|---------|
| **L1 Bitcoin** | PSBT (BIP 174) | Commit + Reveal transaction pair | UniSat, Xverse, Leather |
| **L2 Citrea EVM** | `eth_sendTransaction` | Individual EVM contract call | MetaMask, Brave Wallet, WalletConnect |

These cannot be mixed. A Bitcoin wallet cannot sign an EVM transaction,
and an EVM wallet cannot sign a PSBT. The webapp maintains two separate
wallet slots and routes each action to the correct one.

### Serverless inscription pipeline (L1)

A Bitcoin inscription is not a simple transfer. Every BINST entity
(institution, process template) that lands on Bitcoin L1
requires:

1. **A commit transaction** — funds a Taproot output whose script
   contains the entity JSON inside an Ordinals envelope
2. **A reveal transaction** — spends that Taproot output via the
   script-path, embedding the inscription in the witness data

Both transactions are built **entirely in-browser** by `txbuilder.rs`,
signed by the connected Bitcoin wallet, and broadcast directly to
`https://mempool.space/testnet4/api/tx`.
There is no local `ord` daemon or bridge server.

The Ordinals envelope includes:
- `content-type: application/json` header
- The entity JSON as content
- Tag 3 (`parent`) when a parent inscription ID is set (for hierarchical linking)

When a parent inscription exists, its UTXO is also spent as a second
input of the reveal transaction so Ordinals indexers recognise the
provenance relationship.

### Why individual EVM calls for L2?

EVM transactions are not natively batchable — each call has its own
nonce and `msg.sender` check. Each L2 operation requires its own wallet
pop-up.

---

## Wallet integration

### L1 Bitcoin wallets

L1 wallets are detected via **EIP-6963** (provider announcement) plus
manual `window.*` injection detection as a fallback:

| Wallet | Detection | Signing method |
|--------|-----------|----------------|
| **UniSat** | `window.unisat` | `signPsbt(hex)` |
| **Xverse** | EIP-6963 / sats-connect | `sats-connect` `SignPsbt` request |
| **Leather** | EIP-6963 | `request('signPsbt', …)` |

> **Xverse note:** Xverse exposes two addresses — a payment address
> (P2WPKH, m/84') and an ordinals address (P2TR, m/86'). Inscriptions
> must be funded from the **ordinals** address (`tb1p…`).

### L2 EVM wallets

L2 wallets are detected and connected through the wallet picker modal:

| Wallet | Method |
|--------|--------|
| **MetaMask** | `window.ethereum` (isMetaMask) |
| **Brave Wallet** | `window.ethereum` (isBraveWallet) |
| **WalletConnect** | `@walletconnect/universal-provider` |

### Dual wallet bar

```
[ L1  tb1p…a4f2  ✕ ]  [ L2  0x3f…9a1c  ✕ ]  Citrea Testnet
```

Both slots can be connected simultaneously and disconnected independently.

---

## L1 Stack — institution-aware inscription batching

The **L1 stack** accumulates multiple inscription intents before signing.
When ready, each entry gets its own commit+reveal PSBT pair signed and
broadcast sequentially.

```text
Create Institution ──┐
Design Process ──────┼──→  L1 Stack  ──→  stack_plan.rs  ──→  txbuilder.rs  ──→  wallet sign
Design Process ──────┘    (grouped)       (plan + route)      (PSBT pair)
```

### Stack intelligence layer (`stack_plan.rs`)

`stack_plan::build_plan` analyses the stack before signing and produces
an `ExecutionPlan` — a flat sequence of `PlannedStep`s in broadcast order,
each annotated with exactly how to resolve its parent UTXO:

| Institution state | `ParentSource` |
|---|---|
| Institution is also in this batch | `InBatch { batch_idx }` — live reveal UTXO from same run |
| Institution in mempool (unconfirmed) | `FetchMempool { txid }` — fetch output 0 at sign time |
| Institution confirmed | `FetchMempool { txid }` — same fetch, always works |
| Unknown parent | `None` + warning surfaced before signing |

The stack panel groups entries by institution and shows a state badge
(`in-batch` / `mempool` / `confirmed` / `external`) per group.

### Confirmation polling (`bridge.rs`)

After broadcast, `bridge.rs` polls `https://mempool.space/testnet4/api/tx/{txid}`
until 6 confirmations. The pipeline panel shows only active (unconfirmed
+ < 6 conf) inscriptions and auto-removes entries at 6 confs. Polling
resumes on wallet reconnect via `resume_pending_polls`.

### localStorage registry (`storage.rs`)

Every inscription is persisted in `localStorage` with:
- `reveal_txid` — transaction ID of the reveal tx
- `inscription_id` — Ordinals inscription ID
- `confirmed` flag — updated by the polling loop
- Entity metadata (label, type, parent)

The registry survives page reloads and is the primary source for parent
UTXO lookup during stack planning.

---

## L2 Execute — direct EVM calls

L2 process execution goes through the **Execute view**, not through the
stack. Each EVM transaction fires **immediately** on button click — no
batching, no queue. The L1 stack is Bitcoin-only.

| Action | Contract call | Trigger |
|---|---|---|
| Create Instance | `factory.createInstance(inscriptionId, names[], types[])` | "Create Instance on Citrea" button |
| Execute Step | `instance.executeStep(status, evidence)` | "Execute Step →" button |

All EVM/ABI logic lives in `citrea.rs`. The `execute.rs` module is a
thin UI wiring layer that delegates to `citrea.rs` for every on-chain
call. Instance state (steps, completion, finality) is read directly from
the contract via `eth_call` — no wallet needed for reads.

---

## Module structure

| Module | Purpose |
|--------|---------|
| `lib.rs` | WASM entry point — boots all views |
| `auth.rs` | Authentication state |
| `bridge.rs` | Confirmation polling loop + pipeline panel DOM; `resume_pending_polls` on connect |
| `citrea.rs` | All L2 EVM/ABI logic: `create_instance`, `execute_step`, ABI encoding, finality queries, event log recovery |
| `create.rs` | Create Institution view — form, validates, pushes to stack |
| `decode.rs` | JSON / witness / vault decoder UI |
| `design.rs` | Design Process view — form, fetches parent UTXO, pushes to stack |
| `dom.rs` | DOM helpers, toast notifications, clipboard |
| `effects.rs` | Visual effects (logo tilt, spark effect) |
| `execute.rs` | Execute view — thin UI wiring, delegates to `citrea.rs` for all on-chain calls; finality status display |
| `fetch.rs` | Shared HTTP fetch helpers (Mempool.space API) |
| `inscribe.rs` | Full serverless pipeline: fetch UTXOs → build PSBTs → sign → broadcast; `fetch_tx_output` |
| `institution.rs` | Institution card rendering and navigation |
| `l2_queue.rs` | Pure Rust L2 action queue data model (not wired to UI — kept for tests only) |
| `nav.rs` | View routing and bottom nav, 4 tests |
| `process.rs` | Process view + instance creation via factory flow |
| `search.rs` | Institution search against Ordiscan API + localStorage, 4 tests |
| `stack.rs` | Pure Rust L1 inscription stack — `StackEntry`, ordering, validation, 21 tests |
| `stack_plan.rs` | Pure-Rust execution planner — institution grouping, `InstitutionState`, `ParentSource`, 12 tests |
| `stack_ui.rs` | Stack panel UI — institution-grouped render, plan-driven `on_sign_inscribe` |
| `storage.rs` | localStorage registry: inscriptions, templates, L2 instance tx hashes, 12 tests |
| `txbuilder.rs` | Commit+reveal PSBT construction; parent UTXO as second reveal input, 9 tests |
| `wallet.rs` | L1 BTC wallet — EIP-6963 + manual detection, connect/disconnect, pubkey extraction |
| `wallet_picker.rs` | Wallet picker modal; L2 EVM wallets (WalletConnect, MetaMask, Brave) |

---

## Build

```bash
# Development server (port 8081)
CC_wasm32_unknown_unknown=/opt/homebrew/opt/llvm/bin/clang \
  trunk serve --port 8081

# Release build
CC_wasm32_unknown_unknown=/opt/homebrew/opt/llvm/bin/clang \
  trunk build --release
```

Requires LLVM's clang for the `wasm32-unknown-unknown` target because
the system Apple clang does not support that target.

---

## Test suite

```bash
cargo test  # runs on native target — no browser needed
```

97 tests across 11 modules:

| Module | Tests |
|--------|-------|
| `stack` | 21 |
| `l2_queue` | 13 |
| `storage` | 12 |
| `stack_plan` | 12 |
| `txbuilder` | 9 |
| `decode` | 9 |
| `auth` | 7 |
| `dom` | 6 |
| `search` | 4 |
| `nav` | 4 |
| `institution` | 3 |

UI code (DOM manipulation, wallet calls, async flows) runs only in WASM
and is not unit-tested.
