---
title: "Vendor: Railgun"
status: ready
maturity: production
---

# Railgun - Privacy System for Ethereum and EVM Chains

## What it is

Railgun is a set of smart contracts providing **shielded transfers and private DeFi interactions** on Ethereum and other EVM-compatible chains.  
It uses zk-SNARKs to maintain a UTXO-like note system (commitments and nullifiers) that allows users to send, receive, and interact with DeFi protocols while hiding wallet balances and transaction details.

## Fits with patterns

- [Shielding](../patterns/pattern-shielding.md)
- [Regulatory Disclosure Keys & Proofs](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Proof of Innocence](../patterns/pattern-proof-of-innocence.md)
- [Private ISO 20022](../patterns/pattern-private-iso20022.md)

## Not a substitute for

- Not a cross-chain settlement mechanism (no native atomicity across L1/L2s).
- Not an institutional compliance layer (no built-in regulator viewing keys or scoped audit paths).
- Not a programmable privacy rollup (functions only as contracts on existing L1/L2).

## Architecture

- **Execution model**: UTXO-style commitments (`note commitments`, `nullifiers`) stored in smart contracts.
- **Proof system**: zk-SNARKs (Groth16) validate note creation and spending.
- **Settlement**: ERC-20/ERC-721/ERC-1155 tokens are deposited into Railgun contracts, converted to private notes, then spent/withdrawn.
- **Integration**: Railgun “Adapt Modules” allow private interactions with existing DeFi protocols by wrapping function calls.
- **Data availability**: all commitments stored publicly on-chain; privacy relies on zk proofs, not off-chain DA.

## Privacy domains

- **Shielded transfers**: hide sender, receiver, token, and amount within the anonymity set.
- **Private DeFi calls**: interact with external contracts (e.g. Uniswap) without linking wallet identity.
- **No regulator view keys**: users can self-disclose by sharing spending keys, but no scoped/threshold key system is natively supported.

## Enterprise demand and use cases

- **Retail privacy**: primary adoption among individual users seeking shielded ERC-20 transfers.
- **DeFi traders**: private swaps and interactions with DEXs to prevent MEV or front-running.
- **Institutions**: occasionally cited in research as a shielded pool candidate, but lacks regulator access features required for enterprise settlement.

## Technical details

- zk-SNARK proving via Groth16, circuit for note validity and nullifier uniqueness.
- Smart contract architecture deployed on Ethereum mainnet, BNB Chain, Polygon, and others.
- UTXO model: each note represents a claim on deposited tokens; nullifiers prevent double spends.
- Adapt Modules: special contracts that wrap DeFi calls (e.g. swaps, lending) to preserve privacy of user identity.

## Strengths

- Mature, deployed contracts live on multiple EVM chains.
- Large existing anonymity set (user-driven shielded pool).
- Supports many ERC standards (ERC-20, ERC-721, ERC-1155).
- Extensible via Adapt Modules for DeFi interoperability.

## Risks and open questions

- **No regulator-oriented audit path**: may limit institutional adoption.
- **Prover costs & UX**: Groth16 proofs are relatively heavy for end-users, requiring wallet integration, high gas operations (~2m).
- **On-chain DA**: all commitments are public; long-term scalability may be constrained.
- **Trust assumptions**: requires careful ceremony setup for zk-SNARK parameters (trusted setup).

## Links

- [Railgun GitHub](https://github.com/Railgun-Privacy)
- [Railgun Docs](https://docs.railgun.org/)
- [Railgun Contracts](https://etherscan.io/address/0x3Af3e7A12e1A663C8eEe5F77026F664A0aC1f60C)
- [Railgun Website](https://railgun.org/)
