# Contributing

BINST is an open-source project under the
[Bitcoin-Institutions](https://github.com/Bitcoin-Institutions) organization.

## Repository

- **Pilot:** [github.com/Bitcoin-Institutions/binst-pilot](https://github.com/Bitcoin-Institutions/binst-pilot)
- **Documentation:** [github.com/Bitcoin-Institutions/binst-pilot-docs](https://github.com/Bitcoin-Institutions/binst-pilot-docs)

## Areas of contribution

| Area | Stack | Description |
|------|-------|-------------|
| Smart contracts | Solidity 0.8.24, Hardhat 3 | Extend or improve the four core contracts |
| BINST Protocol | Rust, `no_std` | Improve Bitcoin transaction parsing, add crates |
| Scripts & tooling | TypeScript, Viem | New protocol demonstrations, CLI tools |
| Inscription tooling | Rust, `ord` | Improve `binst` metaprotocol parsing and indexing |
| Webapp | Rust, WASM, Trunk | Improve the pilot web UI |
| Documentation | Markdown, mdbook | Improve or translate this book |
| Testing | TypeScript, Rust | Expand test coverage across all layers |

## Development setup

```bash
# Clone the pilot
git clone https://github.com/Bitcoin-Institutions/binst-pilot.git
cd binst-pilot

# Solidity
npm install
npx hardhat compile
npx hardhat test

# Rust
cd binst-protocol
cargo build
cargo test
```

## License

MIT — see [LICENSE](https://github.com/Bitcoin-Institutions/binst-pilot/blob/main/LICENSE).
