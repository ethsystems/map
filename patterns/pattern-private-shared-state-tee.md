---
title: "Pattern: Private Shared State (TEE)"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-06-18

works-best-when:
  - Multiple institutions share a ledger, pool, or order book and must hide individual positions from each other.
  - Low latency is required and participants accept hardware trust assumptions.
  - Contractual and audit controls can bound hardware-vendor and host-operator risk.
avoid-when:
  - Single-party privacy is sufficient (use shielding instead).
  - The threat model includes nation-state physical access or supply-chain compromise.
  - Full trustlessness is required; prefer the co-SNARK or Fully Homomorphic Encryption variants.

context: both
context_differentiation:
  i2i: "Between institutions, hardware trust is bounded by contractual controls (NDAs, audit rights, attestation logging) and by running enclaves across diverse operators. The consortium can standardise enclave code, measurement values, and rotation policy."
  i2u: "For user-facing deployments a single-operator enclave gives the user no independent verification of integrity. A coalition of host plus hardware vendor can observe plaintext. Threshold keys across multiple independent enclave operators, combined with zero-knowledge proofs of correct execution, are required to make the guarantee meaningful."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Host controls enclave availability and can deny service. Hardware-vendor dependency on attestation roots is unavoidable; users cannot route around a vendor revocation. Remains `low` even with multiple host operators."
  o: "Enclave guest code can be fully open source and reproducibly built. TEE hardware, firmware, and attestation services remain proprietary; drivers and microcode updates are vendor-controlled."
  p: "Plaintext is exposed inside the enclave during execution. Privacy holds only against the host operator and co-tenants; the hardware vendor and a physical attacker with sufficient capability can observe or extract state. Side-channels (cache timing, Spectre-family) remain an active research area."
  s: "Rides on enclave measurement correctness, timely firmware patching, and attestation-chain integrity. Remote attestation catches configuration drift but not silent microcode backdoors."

post_quantum:
  risk: medium
  vector: "Remote attestation typically uses elliptic-curve signatures (ECDSA, EdDSA) broken by CRQC. Recorded attestation reports and sealed blobs face HNDL risk if encrypted to vendor-operated key hierarchies."
  mitigation: "Migrate attestation signing and sealing keys to post-quantum signatures (ML-DSA, SLH-DSA) as vendors ship them. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  requires: [pattern-tee-based-privacy]
  composes_with: [pattern-tee-zk-settlement, pattern-tee-key-manager, pattern-stealth-addresses, pattern-regulatory-disclosure-keys-proofs]
  alternative_to: [pattern-private-shared-state-cosnark, pattern-private-shared-state-fhe]
  see_also: [pattern-shielding, pattern-co-snark]

open_source_implementations:
  - url: https://github.com/oasisprotocol/sapphire-paratime
    description: "Confidential EVM runtime running under TEE with attested execution"
    language: "Rust, Solidity"
  - url: https://github.com/automata-network/multi-prover-avs
    description: "Attested multi-prover infrastructure usable as a TEE execution backend"
    language: "Go, Rust"
---

## Intent

Enable N parties to jointly read and write shared on-chain state (balances, positions, order books, collateral pools) while keeping each party's individual data private from the others. This variant stores state encrypted to an enclave public key; a Trusted Execution Environment decrypts inputs, runs the state transition at native speed, and posts updated encrypted state plus a remote-attestation report proving which code ran.

Unlike single-party shielding, private shared state requires computation across multiple parties' secrets.

## Components

- Trusted execution environment: Intel SGX (process-level), AMD SEV-SNP (VM-level), or AWS Nitro Enclaves (hypervisor-isolated). Each imposes a different threat model and attestation chain.
- Enclave guest code implements the state-transition logic and holds the decrypted state only for the duration of a transition.
- Remote attestation service binds the enclave's measurement (code hash plus configuration) to a signed attestation report that participants and the on-chain verifier can check.
- Commitment scheme (Pedersen, Poseidon) represents state on-chain; an optional zero-knowledge proof hybrid proves correct execution beyond attestation.
- On-chain state store and verifier contract anchor attested state updates; a fallback hybrid path verifies a zero-knowledge proof of execution.
- Regulatory disclosure path uses enclave-mediated decryption scoped to specific positions or time windows.

## Protocol

1. [operator] Deploy the attested enclave cluster; publish enclave measurements and attestation public keys.
2. [user] Each party verifies attestation, then deposits assets; balances are encrypted to the enclave public key.
3. [user] A party submits an encrypted state-transition request to the enclave.
4. [operator] The enclave decrypts inputs, executes the transition at native speed, and emits the updated encrypted state plus an attestation report.
5. [contract] The on-chain verifier checks the attestation (and, in the hybrid variant, a zero-knowledge proof of correct execution) and updates the state root.
6. [auditor] Regulator requests scoped disclosure; the enclave decrypts only the approved scope and emits a signed disclosure record.

## Guarantees & threat model

Guarantees:

- Inputs and intermediate state remain confidential from other parties and from the host operator, subject to the enclave assumptions.
- Remote attestation binds every state transition to a specific code measurement and configuration.
- Settlement finality anchored to Ethereum L1 or an L2.
- Enclave-mediated scoped disclosure for regulators.

Threat model:

- Enclave integrity, including resistance to cache-timing and transient-execution side-channels. Constant-time code and timely firmware patching are required mitigations.
- Hardware-vendor trust: vendor master keys and attestation-chain integrity are unavoidable roots of trust.
- Host-operator availability: the host can schedule or refuse to schedule the enclave; privacy is unaffected, liveness is not.
- Physical access and supply-chain compromise are out of scope for the bare TEE variant; combine with a zero-knowledge execution proof for stronger guarantees.
- Sender and receiver addresses are not hidden by default; address unlinkability requires composition with stealth addresses.

## Trade-offs

- Lowest latency of the private-shared-state variants; execution runs at near-native speed inside the enclave.
- Hardware trust is a hard dependency. Contractual controls and enclave-operator diversity mitigate but do not remove it.
- Attestation-verification gas cost on-chain is non-trivial and varies by platform.
- Side-channel exposure is a moving target; assume ongoing firmware maintenance and code hardening.
- Hybrid designs that add a zero-knowledge proof of correct execution reduce hardware dependence at the cost of proving overhead.

## Example

- Three banks share a tokenised-bond collateral pool on an Ethereum settlement L2. Each bank's deposit is encrypted to the enclave cluster's public key. A margin call triggers enclave computation: the enclave decrypts inputs, evaluates aggregate collateral coverage at native speed, and emits the updated encrypted state with an attestation report. The regulator uses enclave-mediated disclosure to audit one bank's position without learning the others.

## See also
- [Oasis Sapphire ParaTime](https://oasisprotocol.org/sapphire)
- [Intel SGX documentation](https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html)
- [AWS Nitro Enclaves documentation](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html)
- [Inco Lightning](https://docs.inco.org/home)
