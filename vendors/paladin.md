---
title: "Vendor: Paladin"
status: ready
---

# Paladin - Modular System for Programmable Privacy on EVM chains

## What it is
Open-source (Apache-2.0) privacy layer under the [Linux Foundation Decentralized Trust](https://github.com/LFDT-Paladin/paladin). Paladin provides the common wallet/vault functions that are needed to interact with all forms of privacy preserving smart contracts. It also provides a model for atomic programmability across these privacy preserving smart contracts, harnessing the power of the underlying EVM shared ledger.

Paladin uses standard EVM smart contracts as the source of truth for the finalization of the transaction, and verification of the private logic. On-chain state holds masked or commitment state; cleartext states are exchanged off-chain in private channels over mutual TLS or gRPC (or optionally encrypted on-chain).

Reference implementations and samples of smart contracts are provided for asset issuers and holders that need tokenized cash (central bank and commercial bank money), and tokenized assets (bonds and other financial assets), for payments, PvP and DvP.

Runs as a sidecar next to a standard EVM client (for example Hyperledger Besu, or a Layer 2 node like Linea). Zero modifications are required to the underlying blockchain, and any permissioned or public EVM can be used.

A strong design principle of the project is that existing privacy preserving tokens should be able to become compatible with Paladin wallet/vault functions and programmability with limited changes. Three privacy domains are implemented today (see below), with planned future support for other privacy preserving tokens such as confidential ERC20 based on FHE.


## Fits with patterns
- [Shielding](../patterns/pattern-shielding.md)
- [DvP ERC7573](../patterns/pattern-dvp-erc7573.md)
- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md)
- [Crypto Registry Bridge eWpG EAS](../patterns/pattern-crypto-registry-bridge-ewpg-eas.md)

## Not a substitute for
- Privacy L2 or App-chain
- Identity and Attestations

## Architecture
- Sidecar next to any standard EVM node; Paladin acts as a privacy-preserving transaction manager for Ethereum (no client fork).
- Secure channels between Paladin nodes for selective disclosure; data at rest remains private, data in-flight is encrypted end-to-end.
- Privacy-preserving smart contracts split across two parts:
  - Base EVM smart contract on the unmodified chain (ordering/finality, double-spend, proof validation, signature validation).
  - Off-chain Paladin runtime that manages private state, proofs/endorsements, and coordination.
- Provides similar function to legacy privacy-group managers (e.g., Tessera) with additional interoperability and atomic composition across domains.
- Base EVM ledger is the single source of truth; all domains interoperate atomically on one ledger; state stored in masked form to preserve confidentiality and anonymity.
- Transaction manager coordinates assembly, submission, and confirmation to:
  - Public EVM contracts
  - UTXO-based privacy preserving tokens
  - FHE based confidential tokens (future plans)
  - Private EVM contracts in privacy groups
- Key management integrates with enterprise HSM/SSM; supports native Ethereum signing, EIP-712 endorsements, zero-knowledge proof generation, as well as FHE wallet-side cryptography in future plans.
- Programming model includes plug points for:
  - Wallet functions (coin/state indexing and selection for submission)
  - Endorsement coordination and signature collection with flexible endorsement policies
  - Distributed sequencer for transaction coordination
  - Proof generation (zero-knowledge proofs, notary certificates, and others)
  - High-performance code modules in Java and WebAssembly

Privacy domains are implementations of a plug-point for the on-chain logic (pure on-chain EVM), and the app-layer logic (proof systems, merkle tree management, private EVM execution, endorsement/attestation coordination), that are the common components of every privacy-preserving smart contract in the EVM ecosystem.

Paladin aims to be open and flexible enough that any privacy system could plug in - as any transparent ERC-20 can plug into a wallet today.

Hardened reference implementations are provided out of the box as follows:

- Zeto (ZK UTXO tokens)
  - Onchain commitments hide ownership/amounts/history of the UTXOs.
  - Enforces mass conservation and other spending policies (KYC, auditability, etc.) via zero-knowledge proofs.
  - Currently implemented with zkSNARKs in Circom based circuits, using groth16 by default.
  - Paladin runtime includes a token indexer, UTXO selector, and proof generator.
  - Optional ERC20 bridge via deposit/withdraw.
  - Lock/unlock flows prevent proof theft in multi-leg flows (e.g., DvP).
- Noto (notarized UTXO tokens)
  - Onchain commitments hide ownership/amounts/history of the UTXOs.
  - Enforces mass conservation and other spending policies via notary certificates (EIP-712 signatures).
  - Notary/issuer governs confidential UTXO state; can be backed by EOAs or privacy groups.
  - Supports basic and hooks notary modes; private and public ABIs for mint/transfer/burn and lock/unlock flows; approveTransfer and delegateLock enable delegated execution.
- Pente (private EVM execution in privacy groups)
  - Each privacy group is a unique contract that maintains its own private world state as UTXO commitments onchain.
  - Exact transitions are pre-verified off-chain and endorsed; on-chain verification uses threshold/EIP-712 signatures with no base EVM changes.
  - Paladin runs ephemeral Besu EVMs in-process for pre-execution; privacy groups can interoperate atomically with token domains.

## Enterprise demand and use cases
- Segments: asset issuers, asset holders and network builders needing custody/compliance.
- Common implementations: cash-like tokens plus tradeable assets; PvP; DvP; private negotiation of programmable transaction rules.
- Buyer profile: typically VP or Head of Digital Assets; privacy repeatedly cited as a key blocker for public-chain adoption.

## Technical details
- Data transports and registry
  - Two identity types:
    - Account signing identities (long-lived; may be secp256k1, BabyJubJub).
    - Runtime routing identities (for data-in-flight; transport certs/addresses/hosts/topics).
  - Registry plugin resolves routing identifiers to transport details; address book maps friendly names to accounts.
  - Transport principles: asynchronous message transfer, idempotent requests with retries, end-to-end encryption even via hubs/buses.
- Approval-based atomic transactions (DvP/PvP)
  - Pre-approval/setup: parties reach a private state root; prepare token transfers.
  - Approval/prepare: swap contract deployed; each domain pre-approves (privacy group endorsement, notary approval, zero-knowledge proof).
  - Execution/commit: call execute() on the swap contract; all sub-transactions commit or revert atomically.
  - Post-execution: domains remain independent; provenance is hidden except to entitled parties.

## Strengths
- Atomic composition across privacy domains without modifying EVM clients.
- Multiple privacy models under one surface (ZK UTXO, notary-backed UTXO, group-private EVM).
- Enterprise operations alignment: HSM/SSM integration, registry/addressing, predictable governance boundaries.

## Risks and open questions
- Centralization and custodial key management trade-offs in some deployments.
- Operator access to client data in certain operating modes (trust boundary).
- Scope and roadmap for interop with public DeFi and external venues.

## Links
- Architecture overview: https://lfdt-paladin.github.io/paladin/head/architecture/overview/
- Atomic interop (approval-based): https://lfdt-paladin.github.io/paladin/head/architecture/atomic_interop/
- Data transports and registry: https://lfdt-paladin.github.io/paladin/head/architecture/data_and_registry/
- Key management: https://lfdt-paladin.github.io/paladin/head/architecture/key_management/
- Programming model: https://lfdt-paladin.github.io/paladin/head/architecture/programming_model/
- Zeto (ZK tokens): https://lfdt-paladin.github.io/paladin/head/architecture/zeto/
- Noto (notarized tokens): https://lfdt-paladin.github.io/paladin/head/architecture/noto/
- Pente (privacy groups): https://lfdt-paladin.github.io/paladin/head/architecture/pente/
- Repository: https://github.com/LFDT-Paladin/paladin
