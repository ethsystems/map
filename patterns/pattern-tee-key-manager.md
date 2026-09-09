---
title: "Pattern: TEE Key Manager"
status: ready
maturity: testnet
type: standard
layer: offchain
last_reviewed: 2026-06-18

works-best-when:
  - An institution needs hot or warm key custody with stronger isolation than a software-only wallet.
  - The hardware vendor and operator are known, contractually bound, and operationally mature.
  - Signing workloads are moderate in throughput and can tolerate attestation and policy-check latency.
avoid-when:
  - Key security must not depend on specific hardware vendors or cloud operators.
  - The threat model includes strong physical or microarchitectural side-channel attackers.
  - Custody duration exceeds the security lifecycle of the underlying hardware platform.

context: both
context_differentiation:
  i2i: "Between institutions, TEE key managers operate under bilateral agreements covering physical security, attestation verification, TCB recovery, and incident response. Threshold distribution across operators and vendors is expected for anything above pilot value, and attestation measurements become shared artefacts in compliance reviews."
  i2u: "For user-facing keys, a single-operator TEE is insufficient: the user has no independent way to verify enclave integrity, and the operator alone can halt signing. Multi-operator threshold deployments and publishable attestation measurements are required before the custody claim translates to a meaningful user-side guarantee."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "The host operator can refuse to schedule the enclave or drop messages on the I/O channel, so single-operator deployments are structurally censorable. Threshold custody across independent operators raises the floor by requiring coordinated action to withhold signatures."
  o: "Key-manager policy logic and attestation-verification code can be open-sourced, and platforms such as Open Enclave are available. Vendor attestation services and microcode remain closed."
  p: "Private keys stay inside the enclave under the stated trust assumptions. Signing requests and request context are visible to the operator at the channel level unless additionally encrypted end-to-end; this is a metadata leakage concern even when plaintext never leaves the enclave."
  s: "Rides on hardware-vendor key integrity, firmware and microcode discipline, correct enclave code, attestation-verifier correctness, and anti-rollback of protected state. Side-channel and supply-chain risks are active and require continuing platform hygiene."

post_quantum:
  risk: high
  vector: "Signing keys stored inside the enclave (ECDSA on secp256k1, BLS for validator signing) are broken by a CRQC. Attestation chains additionally rely on vendor ECDSA keys (for example, Intel DCAP), so a CRQC invalidates remote attestation itself."
  mitigation: "Post-quantum signature schemes (ML-DSA, SLH-DSA) inside the enclave for application signing, and vendor roadmaps for post-quantum attestation. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  requires: [pattern-tee-based-privacy]
  composes_with: [pattern-verifiable-attestation, pattern-mpc-custody, pattern-user-controlled-viewing-keys]
  alternative_to: [pattern-mpc-custody]
  see_also: [pattern-tee-zk-settlement]

open_source_implementations:
  - url: https://github.com/openenclave/openenclave
    description: "Open Enclave SDK used for enclave-hosted key managers on Intel SGX"
    language: "C, C++"
  - url: https://github.com/fortanix/rust-sgx
    description: "Rust SGX toolchain suitable for building attested enclave services"
    language: "Rust"
---

## Intent

Run an institutional key manager inside an attested hardware-isolated environment. Keys are generated or imported inside the enclave, which enforces a signing policy and produces signatures while keeping private key material isolated from the host and operator. External systems bind the signer public key and policy hash to an approved attested measurement via an attestation verifier; on-chain registries can record this binding for audit.

## Components

- Attested hardware platform (CPU-encrypted enclave or hypervisor-isolated enclave) providing remote attestation and sealed persistence.
- Key-manager workload implementing generation, import, signing policy, and rotation.
- Attestation verification service that validates attestation reports against platform configuration policy and approves signer public keys against a measurement registry.
- Registry that maps approved signer keys to institutions and active policies; can be on-chain via an attestation service or off-chain under stricter controls.
- Internal policy and approvals system feeding rules into the enclave over an authenticated channel.
- Monitoring for enclave health, firmware and TCB status, and attestation freshness.
- Bootstrap path (hardware security module or enterprise key-management service) used only for controlled key import, if legacy keys must be migrated.

## Protocol

1. [operator] Deploy the key-manager workload with an approved initial image and configuration (debug disabled, accepted TCB levels) and obtain an attestation report from the enclave.
2. [verifier] Verify the attestation report, check platform configuration against policy, and register the signer public key and measurement in the registry.
3. [enclave] Generate new keys inside the enclave when the platform provides a strong entropy source, or accept a controlled import over an authenticated channel; store key material using platform-specific protected persistence with explicit anti-rollback.
4. [operator] Load the institution's signing policy (assets, destinations, limits, approvals) into the enclave over an authenticated channel and anchor a hash of the active policy in the registry.
5. [user] Submit signing requests with unsigned transactions or messages plus context (purpose, approvals, limits) to the enclave over an authenticated, encrypted channel.
6. [enclave] Reconstruct signing intent, check the request against policy and protected state (limits, replay, approvals), and either sign or return a rejection without exposing key material.
7. [operator] On key events (creation, import, rotation, policy change), write an append-only record signed by the TEE-held key and anchor its hash in the registry; rotate under a controlled procedure and retire old keys.

## Guarantees & threat model

Guarantees:

- Key confidentiality under the hardware-vendor trust assumptions: the host operating system, hypervisor, and operator cannot read plaintext private keys from the protected execution context.
- Policy-bound signing: the enclave only signs requests that satisfy the attested code and protected policy, assuming correct parsing and strict domain separation (for example, EIP-155 for transactions, EIP-712 for structured messages); avoid "sign arbitrary bytes" interfaces.
- Attested signer identity: approved public keys are tied, via remote attestation, to a specific measurement and platform configuration. Without forging attestation or compromising the registry, an attacker cannot impersonate the key manager.
- Auditable lifecycle: key generation, import, rotation, and policy changes link to approved configurations through the verifier and registry without revealing private keys.

Threat model:

- TEE vendor (silicon, microcode, attestation roots), the operator (physical security, I/O control, availability), and enclave code correctness (parsing, domain separation, policy logic).
- Attestation verifier and registry integrity; compromise of either allows unapproved signer keys to be accepted.
- Supply-chain or firmware compromise breaking TEE isolation or forging attestation.
- Microarchitectural and side-channel attacks leaking keys or policy state until platforms are patched.
- Rollback or snapshot attacks restoring older protected state to bypass limits; anti-rollback and external anchoring are required.
- Bugs in secure boot, attestation checks, debug controls, or policy code that cause over-broad signing while attestation appears valid.
- Key entropy: if the platform does not provide strong entropy to the enclave at generation time, generated keys may not be unique or unpredictable; import or derivation from an external root under controlled processes must be used instead.
- Key confidentiality does not equal key unpredictability. Metadata on signing requests (timing, sizes) is not hidden by this pattern.

## Trade-offs

- Trust anchors (vendor, operator, attestation verifier, registry) are stronger than fully trust-minimised custody and must be reflected in risk assessments.
- Hardware Security Modules remain the baseline for physical tamper resistance; TEE key managers are complementary, not replacements.
- Performance: attestation and policy checks add latency and cap throughput; better suited to treasury, validator, and institutional flows than very high-frequency trading.
- Operational maturity: provisioning, protected-state handling, monitoring, re-attestation, key rotation, and incident response runbooks are all required. Loss or corruption of protected state without a recovery plan can render keys unusable.
- Single-enclave deployments are a single point of compromise. Production custody typically distributes keys across multiple enclaves operated by independent parties; single-enclave deployments should be limited to pilot use or warm operational keys.

## Example

An institution runs a TEE-backed key manager for its treasury and tokenised-asset instruments. Keys are generated inside the enclave on a platform with a strong entropy source; policy restricts transfers to approved counterparties and daily limits. Treasury operators submit unsigned transactions and signed internal approvals over an authenticated channel. The enclave checks the request, signs approved transactions, and returns signatures for broadcast. Key events are anchored in an on-chain registry for audit.

## See also

- [Intel SGX DCAP orientation](https://www.intel.com/content/dam/develop/public/us/en/documents/intel-sgx-dcap-ecdsa-orientation.pdf)
- [AMD SEV-SNP attestation](https://www.amd.com/content/dam/amd/en/documents/developer/lss-snp-attestation.pdf)
- [Ethereum Attestation Service](https://docs.attest.org/)
- [Fireblocks](../vendors/fireblocks.md)
- [Inco](../vendors/inco.md)
