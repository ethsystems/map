---
title: "Pattern: zk-KYC/ML + ERC-734/735 identity claims"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-09-29

works-best-when:
  - Onboarding identities must be publicly verifiable on-chain, for example to enable instant settlement without manual gate-keeping.
  - The issuer wants to prove that KYC or AML was performed under a specific policy without publishing the underlying personal data.
  - A zero-knowledge attestation pipeline (proof of compliant off-chain check, proof of source-of-funds) already exists.
avoid-when:
  - A simple allowlist signed by an authorized compliance party is sufficient and public verifiability is not required.
  - Off-chain auditor access with selective disclosure is preferred over on-chain proof verification.

context: both
context_differentiation:
  i2i: "Between institutions the issuer signing the ERC-734/735 claim is itself regulated. Proof content can be more detailed (policy identifier, screening vendor identifier) because both counterparties can cross-check against contractual obligations."
  i2u: "For users the claim pipeline must not allow the issuer to correlate on-chain activity with the underlying personal data. The zero-knowledge wrapper is what limits issuer over-reach; without it, on-chain attestation gives the issuer a free pseudonym observer. Users also need a clean revocation and re-issuance path when credentials expire."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "The user depends on an issuer to generate the KYC or AML proof. Lifts to `medium` when credentials are portable across competing issuers sharing the same schema."
  o: "ERC-734, ERC-735, and the underlying proof systems are open. Production zero-knowledge machine-learning stacks and some notarized TLS backends are partially open. Issuer tooling and attestation pipelines often ship as closed services."
  p: "The on-chain claim reveals that the subject passed a specific policy check at a specific time. A zero-knowledge wrapper limits the disclosure to the predicate, but the existence and type of claim are visible."
  s: "Rides on soundness of the proof system, integrity of the off-chain computation being proved (for example, a machine-learning score), and issuer key custody. Deployed zero-knowledge machine-learning circuits have been audited only in limited cases."

post_quantum:
  risk: high
  vector: "Pairing-based proof systems used in current zero-knowledge machine-learning stacks (for example, Halo2 with KZG in EZKL) and ECDSA issuer signatures on ERC-735 claims are broken by a CRQC, which allows forged proofs and claims. Any long-lived on-chain claim still relied on when CRQCs arrive loses soundness. The zero-knowledge property does not rest on the broken assumptions, so harvested proofs do not expose the private inputs."
  mitigation: "Migrate the proof backend to hash-based systems (STARK or hash-based SNARK). PQ-safe arithmetization of institutional signature schemes currently imposes a large circuit-size penalty and remains a research frontier. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-734, ERC-735]

related_patterns:
  requires: [pattern-verifiable-attestation]
  composes_with: [pattern-zk-tls, pattern-erc3643-rwa, pattern-regulatory-disclosure-keys-proofs]
  see_also: [pattern-zk-proof-systems, pattern-private-mtp-auth]

open_source_implementations:
  - url: https://github.com/onchain-id/solidity
    description: "ONCHAINID reference implementation of ERC-734/735 (production)"
    language: "Solidity"
  - url: https://github.com/zkonduit/ezkl
    description: "EZKL zero-knowledge machine-learning proving framework, Halo2-based (research/testnet; source-available, no open-source license since January 2024)"
    language: "Rust"
---

## Intent

Publish identity claims on-chain that are backed by a zero-knowledge proof of an off-chain KYC or AML check, instead of a flat signature from a whitelisted issuer. The identity contract (ERC-734/735) holds the claims; a verifier contract checks the zero-knowledge proof at claim ingestion or at access time. The on-chain public sees that a qualified check was performed under a specific policy, without learning the subject's personal data.

## Components

- On-chain identity contract (ERC-734 for key management, ERC-735 for claims) holds the subject's claims and exposes them to gated contracts.
- Claim issuer performs the off-chain KYC, AML, or machine-learning check and generates a zero-knowledge proof attesting that the policy was followed.
- Proof verifier contract checks the zero-knowledge proof against a public policy statement and, on success, writes the claim to the identity contract.
- Source-of-proof integration: either a notarized TLS transcript of the external compliance system, a proof about an existing on-chain state, or a proof over a trusted dataset.
- Policy registry binds policy identifiers to circuit or verifier versions so that verifiers can refuse out-of-date claims.

## Protocol

1. [user] Complete an off-chain KYC or AML check with a qualified provider, or surface the data via a notarized TLS session.
2. [issuer] Generate a zero-knowledge proof that the check passed under a named policy (thresholds, sanctions lists, screening vendor) without revealing the underlying inputs.
3. [user] Submit the proof and the public inputs to the claim verifier contract.
4. [contract] Verify the proof, check the policy identifier against the registry, and write a compliance claim into the subject's ERC-734/735 contract.
5. [contract] Downstream gated contracts read the claim and consult the policy registry to accept or reject the subject.
6. [issuer] Revoke or refresh the claim as the underlying compliance state evolves; the identity contract reflects the current state.

## Guarantees & threat model

Guarantees:

- Public verifiability: any party can check that a qualified compliance check ran under the named policy.
- Minimal on-chain disclosure: only the claim and the policy identifier are public, not the underlying data.
- Instant gating: downstream contracts can settle in a single transaction once the claim exists.
- Issuer accountability: the policy registry ties every claim to a specific circuit and verifier version.

Threat model:

- Soundness of the proof system and correctness of the circuit. A buggy circuit can produce valid proofs for false statements.
- Issuer honesty for inputs the circuit cannot verify independently (for example, the output of an opaque machine-learning model over private inputs).
- Key custody for both the issuer and the identity-contract management keys. A compromised management key can add or remove claims arbitrarily.
- Revocation propagation. A stale claim can persist on-chain after the off-chain compliance state has changed; verifiers must check expiry and the revocation registry.

## Trade-offs

- Proof generation cost, which is substantial for zero-knowledge machine-learning circuits on current hardware. Batch issuance and off-chain delegation of proving reduce user-side cost.
- On-chain verification gas cost is non-trivial; consider doing the verification once at claim ingestion rather than at every access check.
- Circuit evolution is operationally costly. A policy change forces a new circuit and re-issuance for active subjects.
- Importing real-world institutional signatures (for example, government-issued digital-identity signatures) into a zero-knowledge circuit currently carries a large constraint-count overhead.

## Example

A bank onboarding an investor runs a standard KYC and AML check off-chain. The bank's issuer service produces a zero-knowledge proof that the investor passed the bank's published policy. The investor submits the proof to their ERC-734/735 identity contract via the verifier contract, which writes a KYC/AML claim. A tokenized bond contract reads the claim when the investor subscribes and settles the transfer atomically, without ever seeing the investor's personal data.

## See also

- [ERC-734 (unmerged, GitHub issue)](https://github.com/ethereum/EIPs/issues/734)
- [ERC-735 (unmerged, GitHub issue)](https://github.com/ethereum/EIPs/issues/735)
- [EZKL documentation](https://docs.ezkl.xyz/)
- [Approach: Private Identity](../approaches/approach-private-identity.md)
- [Domain: Identity and Compliance](../domains/identity-compliance.md)
