---
title: "Pattern: TEE-Based Privacy"
status: ready
maturity: production
type: standard
layer: offchain
last_reviewed: 2026-06-17

works-best-when:
  - Confidential computation is needed with lower latency than zero-knowledge proofs or multi-party computation can deliver.
  - The deployment can accept hardware and vendor trust anchors in exchange for performance.
  - Operators and hardware vendors are known and contractually bound.
avoid-when:
  - The threat model includes nation-state physical access or supply-chain compromise.
  - Full trust minimisation is required; prefer zero-knowledge or multi-party computation instead.
  - Secrets must outlive the security lifecycle of the hardware platform.

context: both
context_differentiation:
  i2i: "Between institutions, Trusted Execution Environments are operated under bilateral contracts with physical-security controls, audit rights, and SLAs. Threshold distribution across operators and vendors reduces the blast radius of any single compromise. Attestation logs and measurement registries become shared compliance artefacts."
  i2u: "For end users, the institution typically operates the enclave. The user has no independent way to verify the hardware state or the operator's integrity, so any privacy claim rests on institutional trust. Multi-operator or multi-vendor deployments and, where applicable, zero-knowledge proofs of execution, are needed before the user-facing privacy claim has technical substance."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "A single operator that controls scheduling and I/O can deny service at any time; CR is structurally bounded. Multi-operator threshold setups and public measurement registries lift the floor by requiring coordinated action to censor."
  o: "Attestation formats and open-source enclave runtimes are available, but full platform stacks (microcode, vendor attestation services) are closed. Deployment openness depends on how much of the stack the operator publishes."
  p: "Plaintext inside the enclave is hidden from the host operating system and hypervisor under the vendor's trust assumptions. I/O metadata, timing, and microarchitectural side channels remain observable and must be mitigated at the application layer."
  s: "Rides on hardware-vendor key integrity, firmware and microcode patch discipline, and correct enclave code. Documented side channels and firmware vulnerabilities mean the security floor moves with every disclosed CVE."

post_quantum:
  risk: high
  vector: "Attestation chains depend on vendor ECDSA signing keys (for example, Intel DCAP) that a CRQC can forge, invalidating remote attestation. Any long-lived keys sealed inside the enclave using elliptic-curve primitives are similarly exposed."
  mitigation: "Adopt post-quantum signature schemes (ML-DSA, SLH-DSA) for attestation and sealing keys; track vendor roadmaps for post-quantum DCAP. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-tee-key-manager, pattern-tee-zk-settlement, pattern-tee-network-anonymity, pattern-verifiable-attestation, pattern-private-shared-state-tee]
  alternative_to: [pattern-co-snark, pattern-mpc-custody, pattern-private-shared-state-fhe]
  see_also: [pattern-modular-privacy-stack]

open_source_implementations:
  - url: https://github.com/openenclave/openenclave
    description: "Open Enclave SDK supporting Intel SGX and similar hardware TEEs"
    language: "C, C++"
  - url: https://github.com/AMDESE/AMDSEV
    description: "AMD SEV-SNP reference tooling for VM-level attested confidential computing"
    language: "C, Shell"
---

## Intent

Use hardware-isolated execution environments to protect sensitive computation from the host operating system, hypervisor, and operator. The pattern provides hardware-enforced confidentiality and integrity for code and data while running, so privacy-preserving operations can be performed without exposing plaintext to infrastructure providers. This is a foundational pattern; specific applications (key management, settlement, matching) compose on top.

## Components

- CPU-encrypted enclaves (for example, Intel SGX, AMD SEV-SNP) where memory is encrypted by the processor so the host operating system and hypervisor cannot read plaintext.
- Hypervisor-isolated enclaves (for example, AWS Nitro Enclaves) where isolation is enforced by a minimal hypervisor without CPU-level memory encryption.
- Remote attestation services that sign measurement reports rooted in hardware-vendor or cloud-provider keys.
- Measurement registry that records approved code and configuration hashes so verifiers know what "correct" looks like.
- Operational stack: secure provisioning, sealing-key management, firmware and microcode update policy, physical security controls, and incident-response runbooks.

Hardware TEEs differ from Hardware Security Modules. HSMs provide physical tamper resistance (commonly certified to Common Criteria EAL4+), dedicated silicon, and a minimal firmware surface; TEEs offer general-purpose computation with logical isolation but share the CPU die, lack comparable physical tamper resistance, and have a larger attack surface with documented side-channel history. Contractual controls partially mitigate this gap but do not close it; treat TEEs as complementary to HSMs, not replacements.

The two platform families differ in threat coverage:

|                           | CPU-encrypted (SGX, SEV-SNP)            | Hypervisor-isolated (Nitro Enclaves) |
| ------------------------- | --------------------------------------- | ------------------------------------ |
| Protects from             | Host OS, hypervisor, cloud provider     | Parent instance, other tenants       |
| Does not protect from     | CPU manufacturer holding master keys    | Cloud provider controlling hypervisor |
| Memory encryption         | CPU silicon                             | None (hypervisor boundary only)      |
| Attestation root          | CPU-manufacturer signing keys           | Cloud-provider root CA               |
| Side-channel exposure     | High (Spectre, cache timing)            | Lower (VM boundary)                  |

## Protocol

1. [operator] Deploy the workload with a measured code image, generate or import secrets, and configure attestation expectations.
2. [operator] The enclave produces an attestation report containing the code measurement and platform configuration.
3. [verifier] Check the report against the expected measurement and the vendor's certificate chain; bind a session key to a successful attestation.
4. [user] Establish an authenticated channel to the enclave using the bound session key and submit confidential inputs.
5. [enclave] Execute the computation, encrypt outputs before they leave the enclave, and return them over the authenticated channel.
6. [operator] Log attestation events and operations; seal state for persistence with anti-rollback controls.
7. [operator] Rotate code or firmware as required; re-attest and migrate sealed state under a controlled procedure.

## Guarantees & threat model

Guarantees:

- Confidentiality of data and code in memory from the host operating system and hypervisor, under the stated vendor trust assumptions.
- Integrity: attestation proves that the running code and configuration match the approved measurement.
- Auditability: attestation logs create a verifiable execution history.
- Portability: sealed state can be migrated between attested enclaves of the same family.

Threat model:

- Hardware-vendor and microcode correctness, including the attestation-signing key hierarchy.
- Firmware vulnerabilities; exposure is bounded by TCB recovery cadence and update policy.
- Microarchitectural side channels (cache, speculative execution, timing). Constant-time implementations and platform-specific mitigations are required; absence is a live risk.
- I/O-channel control. The host operator controls every byte crossing the enclave boundary and can intercept, reorder, drop, replay, or inject messages even without reading enclave memory.
- Metadata: packet sizes, processing time, and connection patterns remain observable even with encrypted payloads.
- Physical attacks with hardware access, supply-chain compromise, and denial of service (the host can simply refuse to run the enclave).

Failure modes and mitigations:

| Failure mode                           | Impact                                                | Mitigation                                                           |
| -------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------- |
| Supply-chain compromise                | Full confidentiality loss                             | Multi-vendor, hardware audits, attestation checks                    |
| Firmware vulnerability                 | Enclave escape, secret extraction                     | TCB recovery, rapid patching, version policies                       |
| Side-channel attack                    | Partial secret leakage                                | Constant-time code, partitioning, platform mitigations, updates      |
| Rollback attack                        | Policy bypass, double-spend                           | Monotonic counters, external anchoring                               |
| Denial of service                      | Availability loss                                     | Redundancy, fallback procedures, SLAs                                |
| I/O manipulation                       | Data corruption, front-running, censorship            | Authenticated channels with sequence numbers, proofs of execution, multi-operator setups |
| Incomplete attestation verification    | Code substitution while attestation appears valid     | Validate full vendor certificate chain, check platform registers, nonce freshness |
| Key exfiltration from enclave bug      | Full compromise                                       | Audits, formal verification, minimal trusted-computing base           |

## Trade-offs

- Hardware dependency: platforms are tied to specific vendors and subject to supply constraints.
- Trust surface is larger than pure cryptographic solutions; hardware or firmware vulnerabilities can affect all deployments simultaneously.
- Performance: execution is fast but memory is limited in some platforms; context switches add overhead.
- Lifecycle: hardware security erodes as attacks improve, so migration paths must be planned in advance.
- Regulatory acceptance varies; some regulators still treat hardware-isolated execution as opaque for audit purposes.
- Defense layering is usually required: TEE-only deployments are acceptable only for pilots or low-value operations. Production-grade custody and coordination typically combine the TEE with threshold keys and, where verifiability is needed, zero-knowledge proofs of execution.

## Example

A bank deploys a confidential matching engine for block trades inside a hardware-isolated enclave. Traders verify the attestation before submitting encrypted orders; matching runs inside the enclave and only fills are published. Attestation logs record that the approved matching code was executed for each batch, while auditors can verify the code measurement without seeing order flow. Threshold key shares and a zero-knowledge proof of execution are added in a later phase to reduce the blast radius of any single compromise.

## See also
- [Inco](../vendors/inco.md)
- [Confidential Computing Consortium](https://confidentialcomputing.io/)
- [awesome-tee-blockchain](https://github.com/dineshpinto/awesome-tee-blockchain)
- [Fhenix](../vendors/fhenix.md)
- [iExec](../vendors/iexec.md)
