---
title: "Vendor: Aztec Rollup"
status: ready
maturity: production
---

# Aztec - Privacy ZK Rollup

## What it is

Aztec is a privacy focused rollup (or zk-zk-rollup) on Ethereum that enables private transactions and programmable privacy. It is a fully programmable network where applications can access both private and public state.

It uses the **Noir** language along with the Aztec.nr framework to write smart contracts with a hybrid execution model: private functions execute client-side (for privacy), while public functions execute on the network (for transparency/auditability).

## Fits with patterns

- [pattern-noir-private-contracts.md](../patterns/pattern-noir-private-contracts.md) - Noir private smart contracts
- [pattern-privacy-l2s.md](../patterns/pattern-privacy-l2s.md) - Privacy-native rollup execution
- [pattern-shielding.md](../patterns/pattern-shielding.md) - Shielded ERC-20 transfers and confidential balances
- [pattern-shielding.md](../patterns/pattern-shielding.md) - Confidential ERC-20 transfers

## Not a substitute for

- Fully private EVM
- High througput but public rollups

## Architecture

- Hybrid State Model
  - Private state (UTXO-based) is managed by the wallet on the user's device.
  - Public state (Account-based) is managed by the AVM (Aztec Virtual Machine) on nodes.
- Smart contracts are written in [Noir](../patterns/pattern-noir-private-contracts.md) using the Aztec.nr framework.
- Proof system: Honk (UltraHonk) and UltraPlonk. Honk allows for fast recursion and removes the need for a trusted setup.
- DA model: Rollup posts data to Ethereum L1 using EIP-4844 Blobs.
- Settlement: Decentralized sequencers; L2 validity proofs are verified on Ethereum L1.

## Privacy domains

- **Private transfers**: Optional shielding of token amounts, counterparties from the public chain.
- **Selective disclosure**: Users can export viewing keys for auditors/regulators.
- **Programmable privacy**: Circuits allow private execution of DeFi-like logic (DEX, lending) within Aztec.

## Enterprise demand and use cases

- Financial institutions: private stablecoin transfers and settlement.
- Asset managers: confidential DeFi strategies and portfolio movements.
- Corporate treasuries: cross-border payments with regulatory audit but hidden competitive data.

## Technical details

- zkSNARKs: Plonkish proving system with efficient verifier contracts.
- UTXO note commitments with nullifiers to prevent double spends.
- Wallets discover notes via a [note tagging mechanism](https://docs.aztec.network/developers/docs/foundational-topics/advanced/storage/note_discovery#note-tagging) to avoid having to trial-decrypt all note history.
- L1/L2 communication relies on "Portals". These are pairs of contracts (one on Ethereum L1, one on Aztec L2) that pass messages asynchronously via the rollup contract, enabling token bridges and cross-chain governance without trusted 3rd parties.
- Native account abstraction at the protocol level; all accounts are smart contracts.
- Decentralized block building. Validators stake AZTEC tokens to participate in block production. A proposer is chosen per L2 slot at random through RANDAO, and a committee must attest to the validity of the blocks it produces.
- An escape hatch mechanism allows users to exit the chain even if the validator set censors them.

## Strengths

- Strong privacy guarantees for: any private data, private function execution, private smart contract code, privacy over who executed the functions.
- Programmable privacy smart contract execution extend beyond simple shielded transfers.
- Mature research team with open-source infrastructure and audits.

## Risks and open questions

- State Storage, user wallets must store their entire private state to be able to access it, rather than just querying a node.
- Client-Side Proving, private execution requires local execution and proof generation (via PXE), demanding compute resources for end users (eg an AMM swap takes 8s to prove in an Android device).
- Compliance vs. Permissionlessness, While "Selective Disclosure" exists, it is unclear if regulators will accept retroactive auditing over proactive censorship (e.g., OFAC lists at the sequencer level).
- Performances, this system requires a lot of engineering at the cost of a lower throughput, raising the question of use cases that it could tackle.

## Links

- [Aztec Docs](https://docs.aztec.network/)
- [Aztec GitHub](https://github.com/AztecProtocol)
  - [Zk Backend](https://github.com/AztecProtocol/aztec-packages)
- [Aztec Medium: Rollup Architecture](https://medium.com/aztec-protocol)
- [Aztec Connect Overview](https://aztec.network/connect)
