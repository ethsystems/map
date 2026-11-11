---
title: "Vendor: iExec"
status: ready
maturity: mainnet
---

# iExec – Confidential Computing & Off-Chain Execution (TEE-Based Verifiable Compute for EVM)

## What it is

iExec provides privacy-preserving computation, data governance, and trusted execution for decentralized applications through a decentralized network of **Trusted Execution Environments (TEEs)**.

## Fits with patterns

- [Pattern: TEE-Based Privacy](../patterns/pattern-tee-based-privacy.md) - TEE-backed tasks execute securely off-chain with attested results on-chain; core fit for enclave-based compute.
- [Pattern: Attestation verifiable on-chain](../patterns/pattern-verifiable-attestation.md) - Cryptographic attestation proves expected code ran in a genuine enclave; verified results are consumed on-chain.
- [Pattern: Hybrid TEE + ZK Settlement](../patterns/pattern-tee-zk-settlement.md) - Private settlement and matching inside TEEs with ZK proofs for on-chain verifiability; compatible with hybrid TEE + ZK approaches.
- [Pattern: TEE key manager](../patterns/pattern-tee-key-manager.md) - Keys and secrets are used inside secure enclaves, supporting confidential execution models.
- [Pattern: Pre-trade privacy (encrypted order flow, private RFQ)](../patterns/pattern-pretrade-privacy-encryption.md) - Sealed-bid and private RFQ flows; bids processed confidentially off-chain in TEEs, with settlement revealed on-chain.
- [Pattern: Low-cost L2 + Off-chain Encrypted Audit Log](../patterns/pattern-l2-encrypted-offchain-audit.md) - Off-chain encrypted execution with verifiable audit trail; compatible with confidential compute and attested logs.
- [Pattern: Permissioned Ledger Interoperability](../patterns/pattern-permissioned-ledger-interoperability.md) - Bridges enterprise systems and permissioned ledgers with confidentiality and compliance using TEE-based workflows.

## Not a substitute for

- Fully Homomorphic Encryption (FHE)-based on-chain privacy (e.g., fhEVM)
- ZK-based shielded L2s (e.g., Aztec)
- Pure MPC-based decentralized compute networks

## Architecture

Smart contracts emit task requests that are matched via the iExec marketplace. A worker node executes the task inside a **Trusted Execution Environment**. A cryptographic attestation proves the expected code was executed inside a genuine secure enclave; the verified result is returned on-chain.

Core components include PoCo (Proof-of-Contribution), TEE-enabled worker nodes (Intel SGX), a decentralized marketplace for compute, data, and applications, on-chain verification.

## Privacy domains

- TEE-based confidentiality: data is decrypted inside secure enclaves and not exposed in plaintext to infrastructure operators.
- Code confidentiality: application logic can be protected and executed privately within enclaves.
- Hybrid models combining TEE execution with ZK proofs for verification, on-chain settlement, off-chain AI pipelines, and enterprise compliance.

## Enterprise demand and use cases

- Financial institutions and DeFi applications requiring **verifiable off-chain confidentiality** with attested settlement.
- Confidential portfolio computation, risk analysis, private liquidations, and sealed-bid auctions.
- AI and data marketplaces enabling secure dataset monetization and confidential AI inference.
- Web3 and Web2 integration through secure SaaS automation, enterprise data bridges, and confidential API orchestration.

## Technical details

- Ethereum-compatible, live on mainnet
- Intel SGX-based TEE infrastructure
- JavaScript / TypeScript SDKs and smart contract integration tooling
- Decentralized task marketplace

## Strengths

- Infrastructure designed for production deployments
- Hardware-backed TEE security model
- Verifiable off-chain execution
- Integration possibilities across AI, data, and Web3 applications
- Enterprise and Web2 interoperability capabilities

## Risks and open questions

- Reliance on hardware security assumptions (Intel SGX trust model)
- Centralization considerations around TEE hardware supply
- Workloads requiring GPUs or specialized hardware accelerators cannot run inside SGX enclaves
- Very large memory workloads are constrained by enclave memory limits

## Links

- [https://iex.ec](https://iex.ec)
- [https://github.com/iExecBlockchainComputing](https://github.com/iExecBlockchainComputing)
- [https://docs.iex.ec](https://docs.iex.ec)
