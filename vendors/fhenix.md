---
title: "Vendor: Fhenix"
status: ready
maturity: testnet
---

# Fhenix - FHE Based CoProcessor for EVM chains (Fully Homomorphic Encryption as a Service)

## What it is

Fhenix builds a CoProcessor that brings Fully Homomorphic Encryption (FHE) to EVM chains, enabling developers to compute directly on encrypted data without ever decrypting it. This allows dApps to offer on-chain privacy by default. At its core, Fhenix introduces CoFHE, a decentralized coprocessor that makes encrypted computation fast, scalable, and easy for Solidity developers to adopt.

## Fits with patterns

- [Private Stablecoin Shielded Payments](https://github.com/ethsystems/map/blob/master/patterns/pattern-private-stablecoin-shielded-payments.md) - FHE enables encrypted balances and amounts, supporting fully confidential stablecoin transfers.
- [Private Intent-Based Vaults](https://github.com/ethsystems/map/blob/master/patterns/pattern-private-vaults.md) - FHE-powered execution allows vault strategies to run privately while preserving on-chain auditability of assets.
- [Shielded ERC-20 Transfers](https://github.com/ethsystems/map/blob/master/patterns/pattern-shielding.md) - Native FHE support provides encrypted ERC-20 transfer flows for end-to-end confidentiality.
- [Confidential ERC-3643](https://github.com/ethsystems/map/blob/master/patterns/pattern-erc3643-rwa.md) - FHE makes it possible to extend ERC-3643 into a confidential standard with private, enforceable compliance logic.
- [Private L2s](https://github.com/ethsystems/map/blob/master/patterns/pattern-privacy-l2s.md) - Fhenix’s CoFHE serves as an FHE coprocessor enabling privacy-preserving rollups and encrypted execution environments.

## Not a substitute for

- ZK-based L2 privacy (e.g., Aztec, Scroll)
- MPC or TEEs for off-chain privacy computation

## Architecture

CoFHE, Fhenix’s encrypted-computation coprocessor, enables smart contracts to perform arithmetic directly on encrypted values, bringing confidential computation seamlessly into standard EVM workflows.

## Privacy domains

- Fully Homomorphic Encryption at the contract and variable level.
- Supports hybrid models (FHE + ZK for verification).

## Enterprise demand and use cases

- Financial institutions: [private stablecoin](https://github.com/ethsystems/map/blob/master/use-cases/private-stablecoins.md) transfers, on-chain strategies management and [RWA tokenization](https://github.com/ethsystems/map/blob/master/use-cases/private-rwa-tokenization.md).
- Treasury managers: confidential DeFi strategies, conditional orders and portfolio management.
- Private or dual-mode stablecoins, on-chain private intents, confidential lending markets and privacy-preserving yield strategies. 

## Technical details

- EVM-compatible FHE runtime (`FHE.sol`)
- SDKs for Solidity and TypeScript
- Ongoing collaborations with Zama

## Strengths

- Native EVM integration
- Strong security guarantess
- Easy integration for dApps

## Risks and open questions

- High compute cost and latency
- Cross-network interoperability for FHE systems is still developing

## Links

- https://fhenix.io/
- https://cofhe-docs.fhenix.zone/
- https://github.com/fhenixprotocol/
