---
title: "Vendor: Miden"
status: ready
maturity: PoC
---

# Miden – Privacy Rollup

## What it is

Polygon Miden is a privacy focused Ethereum rollup (zk-zk-rollup) that prioritizes throughput and privacy over full EVM compatibility. It uses the Miden VM, a STARK-based virtual machine designed for client-side proving.

Unlike Ethereum (where the network executes everything), Miden pushes execution to the user (the "Edge"). Users execute their own transactions locally, generate a zero-knowledge proof, and the network simply verifies the proof and updates the state.

## Fits with patterns

- [pattern-shielding.md](../patterns/pattern-shielding.md) - Confidential balances and shielded transfers
- [pattern-private-iso20022.md](../patterns/pattern-private-iso20022.md) - Private messaging & settlement
- [pattern-privacy-l2s.md](../patterns/pattern-privacy-l2s.md) - Privacy-native rollup execution

## Not a substitute for

- Fully private EVM
- High througput but public rollups

## Architecture

- Execution model: Actor Model (Concurrent). Unlike the EVM (sequential global state), every `Account` and `Note` is an isolated "Actor" with a local state. Transactions between independent accounts can be executed and proven in parallel, as they don't require locking shared global state.
- Hybrid State Model
  - Accounts hold persistent state (like a wallet or DeFi pool).
  - Notes (UTXOs) carry assets and scripts between accounts.
- Smart contracts are written in Rust (compiling to Miden Assembly/MASM).
- Proof system: zk-STARKs (via Winterfell). Quantum-secure, transparent (no trusted setup), and optimized for recursion.
- DA model: Rollup posts data to Ethereum L1 (utilizing EIP-4844 blobs).
- Settlement: L2 validity proofs are verified on Ethereum L1.

## Privacy domains

- **Private transfers**: Default shielding of token amounts, counterparties hidden from public chain.
- **Programmable confidentiality**: Hybrid model enables both public and private state.
- **Client-side execution**: Users execute transactions locally and submit proofs, keeping transaction details private from public but efficiently provable.

## Enterprise demand and use cases

- Financial institutions: private stablecoin transfers and settlement.
- Asset managers: confidential DeFi strategies and portfolio movements.
- Corporate treasuries: cross-border payments with regulatory audit but hidden competitive data.

## Technical details

- A "transfer" is creating a Note. The recipient must execute a transaction to "consume" the Note. Notes carry their own scripts (e.g., "Only consumable if Oracle X says price > $100").
- The user _is_ the prover, from its own client or through delegated proving. This allows for infinite horizontal scaling because the network does not re-execute complex logic, it only verifies the proof.
- A high-performance STARK prover (Winterfell) used to generate proofs for the Miden VM.
- L1/L2 communication bridging still to be defined.
- Native account abstraction at the protocol level; accounts are smart contracts with updatable code.
- Because users generate the proofs, the Sequencer is lightweight: it only aggregates proofs and builds blocks, preventing the "bottleneck" seen in EVM rollups.

## Strengths

- Massive Concurrency: Parallel transaction processing prevents "gas wars" between unrelated apps, resulting in privacy with high throughput.
- Privacy by Design: Local execution naturally hides user data without complex "add-on" privacy mixers.
- Quantum Security: Relies on hash-based STARKs.

## Risks and open questions

- Audit/Disclosure, path for regulators still unclear.
- Developer Friction, high learning curve (Rust/MASM + Actor model vs. Solidity/EVM).
- Data Availability, if a user loses their private local state (and didn't back it up), they may lose access to their Private Account.
- Wallet Complexity, Wallets must be "smart" enough to track, discover, and consume Notes automatically for a good UX. Client-side proving requires either local compute resources or delegation to a proving service.

## Links

- [Polygon Miden Docs](https://docs.polygon.technology/miden/)
- [Polygon Miden GitHub org](https://github.com/0xMiden)
- [miden-base (core components)](https://github.com/0xMiden/miden-base)
- [crypto (hashes & primitives)](https://github.com/0xMiden/crypto)
- [Note Types (public/private/encrypted)](https://docs.polygon.technology/learn/miden/note_types/)
- [Polygon Miden Alpha Testnet v6 blog](https://blog.polygon.technology/polygon-miden-alpha-testnet-v6-is-live/)
- [Awesome Miden (community resources)](https://github.com/phklive/awesome-miden)
