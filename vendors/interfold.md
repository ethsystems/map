---
title: "Vendor: The Interfold"
status: draft
maturity: testnet
---

# The Interfold – Distributed Network for Confidential Coordination (E3 Protocol)

## What it is

The Interfold is an open-source protocol that coordinates **Encrypted Execution Environments (E3s)** through a distributed network of ciphernodes. It enables independent parties to compute over encrypted inputs and produce shared, verifiable outcomes without concentrating custody, visibility, or execution authority in a single operator. The protocol integrates Fully Homomorphic Encryption (FHE), Zero-Knowledge Proofs (ZKPs), and Distributed Threshold Cryptography (DTC), along with blockchain-based protocol economics, to ensure correctness and confidentiality throughout execution.

## Fits with patterns

- [Publicly Verifiable DKG & Threshold Decryption](../patterns/pattern-verifiable-dkg-threshold-decryption.md) — core cryptographic layer: PVSS-based DKG with on-chain verifier circuits, threshold decryption with per-share ZK proofs, and slashing for provable misbehavior
- [Ephemeral Committees & Disposable Encrypted State](../patterns/pattern-ephemeral-committees.md) — single-use committees with mandatory key disposal after decryption, rendering all undecrypted state permanently inaccessible
- [Private Shared State (FHE)](../patterns/pattern-private-shared-state-fhe.md) — FHE-based computation across multiple independent parties
- [Private Set Intersection (FHE)](../patterns/pattern-private-set-intersection-fhe.md) — encrypted computation over datasets from multiple sources
- [Pretrade Privacy (Encryption)](../patterns/pattern-pretrade-privacy-encryption.md) — sealed-bid auctions and confidential matching
- [Threshold Encrypted Mempool](../patterns/pattern-threshold-encrypted-mempool.md) — shared threshold cryptography approach for distributed trust
- [TEE-Based Privacy](../patterns/pattern-tee-based-privacy.md) — E3s are an alternative to TEEs, replacing trusted hardware with distributed threshold cryptography
- [Noir Private Contracts](../patterns/pattern-noir-private-contracts.md) — Interfold uses Noir for ZK circuit (C0–C7) development

## Not a substitute for

- ZK-based L2 privacy (e.g., Aztec, Scroll) — Interfold coordinates ephemeral computations, not persistent shielded state
- General-purpose MPC — Interfold targets specific multiparty coordination use cases with FHE-based execution
- TEEs — Interfold replaces hardware trust with cryptographic and economic guarantees, not hardware enclaves

## Architecture

The protocol coordinates five lifecycle phases per E3:

1. **Request**: A Requester submits a computation request specifying the E3 Program, committee size, input window, and Compute Provider parameters. A fee proportional to requested resources is deposited.
2. **Node Selection**: Ciphernodes are selected via verifiable sortition (weighted by ticket balances) to form a Ciphernode Committee (CiCo). The committee performs distributed key generation (DKG) and publishes a shared public key.
3. **Input Window**: Data Providers encrypt inputs to the CiCo's public key and submit them on-chain with ZKPs proving validity (correct encryption, valid format, non-reuse).
4. **Execution**: A Compute Provider runs the Secure Process (FHE computation) over encrypted inputs and publishes the ciphertext output with a proof of correct execution. Supported backends include RISC Zero zkVM.
5. **Decryption**: The CiCo collectively performs threshold decryption of the output and publishes the plaintext result. Rewards are distributed to committee members on completion.

**Smart contract suite** (EVM, deployed on Sepolia testnet):
- **Interfold Contract** — central coordinator managing E3 lifecycle, committee selection, and Merkle-tree input integrity
- **E3 Program (E3P) Contract** — defines computation logic, validates inputs, and verifies execution proofs
- **Decryption Verifier** — validates threshold decryption shares
- **Bonding Registry** — manages FOLD token license bonds, tFOLD ticket balances, registration, and exit queues
- **Slashing Manager** — processes slashing for misbehavior (attestation-based and evidence-based lanes with appeal windows)
- **E3 Refund Manager** — calculates and distributes refunds on E3 failure

**Compute Providers** support multiple trust models: verifiable (RISC Zero, with SP1 and Jolt planned) and oracle-based (zkTLS VMs, committee-based, game-theoretic — all coming soon).

## Privacy domains

- **FHE**: BFV scheme via `fhe.rs` and the SAFE library for computation on encrypted data
- **Threshold cryptography**: PVSS-based distributed key generation; decryption requires a threshold of ciphernodes
- **ZKPs**: Noir circuits for input validation (C0–C7 phases) and proof of correct execution
- **Ephemeral keys**: Single-use committee keys eliminate long-lived key material risks — keys are treated as toxic waste after decryption

## Enterprise demand and use cases

The Interfold targets three coordination categories:

- **Competitive coordination**: sealed-bid auctions, RFQ desks, and allocation mechanisms where bid strategy must remain hidden until the outcome is finalized — without granting the operator privileged visibility.
- **Collective coordination**: secret ballots, governance voting, and committee selection where private preferences must produce verifiable, coercion-resistant results without a trusted tallying authority.
- **Data coordination**: cross-institutional analysis (medical research, financial risk modeling, collaborative AI training) where sensitive datasets from multiple organizations contribute to a shared result without pooling or custodial transfer of raw data.

**Concrete implementations**: The CRISP (Coercion-Resistant Impartial Selection Protocol) is Interfold's flagship application — an Aragon plugin enabling secret ballot governance for DAOs with voter registry, key switching, and coercion resistance (building on MACI concepts but replacing the single trusted coordinator with distributed ciphernodes).

## Technical details

- **Networks**: Sepolia testnet and Ethereum mainnet (both deployed)
- **Cryptography**: Threshold BFV FHE, PV-TBFV key generation, Noir ZK circuits (C0–C7 phases), RISC Zero zkVM for verifiable compute
- **SDK**: Interfold SDK for Rust, with Noir circuit toolchain (`interfold noir` CLI)
- **Tokenomics**: FOLD (license bonding, governance), tFOLD (non-transferable ticket token for sortition), USDC for ticket purchases
- **Ciphernode requirements**: 100 FOLD license bond minimum, 1+ ticket (USDC-backed), 8+ cores / 32GB+ RAM / 500GB+ NVMe
- **Timeout configuration** (Sepolia): Committee formation 3,600s, DKG 7,200s, Compute 86,400s, Decryption 3,600s

## Strengths

- **No single point of execution control**: threshold cryptography separates key generation, computation, and decryption across independent ciphernodes
- **Economic security**: staking, slashing, and refund mechanisms incentivize honest behavior and penalize misbehavior
- **Modular Compute Provider model**: supports both verifiable (ZK) and oracle-based execution backends
- **Ephemeral execution**: single-use keys and bounded E3 sessions minimize attack surface
- **Open-source**: full protocol and toolchain publicly available, built by Gnosis Guild
- **Coercion resistance**: CRISP demonstrates practical coercion-resistant voting without a trusted coordinator

## CROPS profile

| Product | CR | OS | Privacy | Security | Context |
|---------|----|----|---------|----------|---------|
| Interfold (E3 Protocol) | high | yes | high | high | both |

## Risks and open questions

- Production alpha network is live; real-world throughput and long-term network decentralization remain to be proven at scale
- FHE compute cost and latency may limit applicability for time-sensitive or high-volume use cases
- For verifiable Compute Providers (RISC Zero), anyone can run the program over published inputs and generate a valid proof — the CP is an on-chain verifier contract with no single party that can refuse to publish or halt the E3. For oracle-based providers (planned), trust assumptions will differ

## Links

- [The Interfold](https://theinterfold.com)
- [Developer Documentation](https://docs.theinterfold.com)
- [Confidential Coordination Blog Series](https://blog.theinterfold.com/tag/confidential-coordination/)
- [Whitepaper](https://docs.theinterfold.com/whitepaper)
- [Ciphernode Operator Guide](https://docs.theinterfold.com/ciphernode-operators)
- [CRISP (Secret Ballot Reference Implementation)](https://docs.theinterfold.com/CRISP/introduction)
