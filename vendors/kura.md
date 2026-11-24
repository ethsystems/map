---
title: "Vendor: Kura"
status: draft
maturity: testnet
---

# Kura — Programmable Private Settlement for Enterprises

## What it is

Kura is built on smart contracts for Ethereum and other EVM-compatible chains to support private payments for enterprises and financial institutions. It combines zero-knowledge asset transfers with issuer and service-provider policies, enterprise wallet approvals, and disclosure to auditors. It focuses on business-to-business stablecoin settlement and integrates with wallets and payment applications. Consumer shielded transfers are also supported.

## Fits with patterns

- [Shielding](../patterns/pattern-shielding.md)
- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md)
- [Regulatory Disclosure Keys & Proofs](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Safe Proof Delegation](../patterns/pattern-safe-proof-delegation.md)

## Not a substitute for

- **A standalone privacy L1/L2**: Kura operates as smart contracts on Ethereum and other EVM-compatible chains.
- **Identity verification, sanctions screening, or audit operations themselves**: Kura incorporates verification and screening results into transaction policies and supports selective disclosure for audits.

## Architecture

- **Assets**: Registered tokens enter the contract and become UTXO-style notes. Cryptographic hash commitments to notes are stored in a Merkle tree; nullifiers prevent double spending.
- **Verification and settlement**: Proofs validate value conservation and wallet, application, and policy authorizations. Multiple transfers and state updates can settle atomically within one operation in the same deployment.
- **Transaction controls**: Issuer and service-provider policies govern eligibility, counterparties, and transfer routes. Policies can suspend eligibility and authorize asset recovery by designated executors. Required checks follow actual transfer paths and cannot be omitted.
- **Independent operators**: Multiple service providers can share a Kura contract, each setting its own eligibility requirements, accepted assets, and counterparties. Transfers between users of different providers must satisfy both providers' conditions.
- **Kura Service**: An off-chain component supporting the use of Kura. It provides wallets and payment applications with APIs for balance and history queries, transaction preparation, proof generation, and transaction submission. Proof delegation alone need not grant spending authority.
- **Data availability**: Each deployment fixes either on-chain publication of encrypted notes and transfer records, or hash publication with external storage.

## Privacy domains

- **Private transfers**: Protect internal counterparties, assets, amounts, and balances from public chain observers. Public deposits, withdrawals, external calls, executors of policy-authorized asset recovery, timing, and counts leave observable information.
- **Audit disclosure**: Viewing and spending authority are separate. Issuers, service providers, and auditors can receive access scoped to configured assets and customers. Complete history reconstruction requires policies covering the relevant routes, viewing/recovery keys, and retained history.

## Enterprise demand and use cases

- **Enterprises and payment providers**: Business-to-business stablecoin settlement, treasury operations, payroll, and supplier payments.
- **Banks and asset issuers**: KYC/KYB and allowlist-based transfer controls, audit disclosure, and policy-authorized asset recovery.
- **Settlement platforms**: Applications combining conditional payments, swaps, and escrow with enterprise wallet approvals.
- **Wallet providers**: API integration for shielded transfers by corporate and individual users, subject to issuer and service-provider transaction policies.

## Technical details

- **Proof system**: A zk-STARK-based zkVM proves off-chain program execution and recursively composes the required proofs. A Groth16 zk-SNARK wrapper provides on-chain verification.
- **Private applications**: Programs define authorization rules, business logic, and state transitions. Zero-knowledge proofs verify execution over private transaction data and application state, enabling multisignature approvals, spending limits, conditional settlement, and other applications.
- **Existing wallet integration**: Existing Ethereum EOA signing keys can also authorize private payments in Kura. Contract wallets can fund and manage linked Kura accounts. Integrations that synchronize signers and approval thresholds enable private payments under the existing multisignature structure.
- **Encryption and history recovery**: Public-key-based key establishment and authenticated encryption allow recipients and authorized recovery recipients to decrypt notes and transfer records. Viewing keys alone cannot authorize spending.

## Strengths

- Brings private payments into enterprise workflows using the signers and approval thresholds of existing contract wallets.
- Separates issuer requirements from operator policies, combining privacy with transaction controls and audit disclosure.
- Extends payments into stateful private applications on a shared asset and verification foundation.

## Risks and open questions

- **Quantum computing and long-term confidentiality**: Stored encrypted transfer records face the risk of future decryption by quantum computers. Separating viewing keys by party and limiting the distribution of viewing public keys can reduce the scope of exposure. Deployments can also publish only ciphertext hashes on-chain, exchanging the records through private channels and storing them privately off-chain. These measures reduce information exposure; they do not guarantee that the cryptography itself is quantum-resistant.
- **Disclosure during delegated proving**: When proof generation is delegated through Kura Service, the transaction information supplied may be disclosed to its operator. Using a TEE to isolate transaction information from the operator, with protection covering input handling and logs, can mitigate this risk.

## Links

- [Kura website](https://kura.datachain.jp)
- [Kura docs](https://docs.kura.datachain.jp/en/)
