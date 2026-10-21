---
title: "Approach: Private Identity"
status: ready
last_reviewed: 2026-10-21

use_case: private-identity
related_use_cases: [resilient-identity-continuity]

primary_patterns:
  - pattern-private-mtp-auth
  - pattern-zk-kyc-ml-id-erc734-735
  - pattern-zk-tls
  - pattern-verifiable-attestation
  - pattern-voprf-nullifiers
supporting_patterns:
  - pattern-regulatory-disclosure-keys-proofs
  - pattern-co-snark
  - pattern-stealth-addresses
  - pattern-erc3643-rwa
  - pattern-compliance-monitoring
  - pattern-network-anonymity
  - pattern-noir-private-contracts
  - pattern-tee-based-privacy
  - pattern-zk-wrappers
  - pattern-social-recovery

open_source_implementations:
  - url: https://github.com/semaphore-protocol/semaphore
    description: "Semaphore (Merkle membership zero-knowledge proofs)"
    language: TypeScript / Circom
  - url: https://github.com/zkpassport
    description: "ZKPassport (NFC passport, Noir circuits)"
    language: TypeScript / Noir
  - url: https://github.com/anon-aadhaar
    description: "Anon Aadhaar (Indian national ID, RSA in ZK)"
    language: TypeScript / Circom
  - url: https://github.com/0xPARC/pod2
    description: "POD2 (composable Provable Object Datatypes, Plonky2 backend)"
    language: Rust
  - url: https://github.com/tlsnotary
    description: "TLSNotary (zk-TLS transcript proofs)"
    language: Rust
  - url: https://github.com/zkproofport/proofport-app
    description: "ZKProofport (mobile/on-device ZK predicates over signed attestation credentials using Noir, UltraHonk, and mopro, with EVM verification)"
    language: TypeScript / Noir

pocs:
  folder: pocs/private-identity
  requirements: pocs/private-identity/REQUIREMENTS.md
  pocs:
    - name: "Resilient Private Identity"
      sub_approach: "Issuer-Independent Enrollment via Distributed OPRF"
      spec: pocs/private-identity/resilient-private-identity/SPEC.md
      status: implemented
---

# Approach: Private Identity

## Problem framing

### Scenario

A bank needs to verify that counterparties and end users meet KYC, sanctions, and jurisdictional eligibility requirements without exposing personal data on chain or building cross-institutional dossiers. The same architecture must serve government registries, DAOs, and humanitarian programs whose users may be in jurisdictions where the issuer becomes unavailable, hostile, or destroyed.

### Requirements

- Verify eligibility claims without revealing the underlying identity
- Composable across credential sources (passport, national ID, email, web2 data, community attestation)
- Sybil resistance with per-identity cost curve (legitimate pseudonyms cheap, mass sybils expensive)
- Selective disclosure for regulators with scoped, time-bound access
- Issuer-independent verification path: on-chain trust anchor only, no issuer contact during verification

### Constraints

- Open-source verification logic; no canonical identity provider
- Proof generation practical on consumer hardware (mobile + browser)
- Revocation without re-identification
- Recovery from device loss or guardian compromise
- Coercion resistance for governance and humanitarian contexts

## Approaches

### Registry-Based Membership Proofs

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: yes, p: full, s: high }
uses_patterns: [pattern-private-mtp-auth, pattern-zk-kyc-ml-id-erc734-735, pattern-regulatory-disclosure-keys-proofs, pattern-voprf-nullifiers]
example_vendors: []
```

**Summary:** A registry operator maintains a Merkle tree of approved members; provers generate ZK membership proofs without revealing which leaf they correspond to.

**How it works:** The operator publishes an on-chain Merkle root commitment with an incremental tree. Members generate zero-knowledge proofs of inclusion in the membership tree and exclusion from a revocation tree, with scope-bound nullifiers preventing replay across verifiers. Contract hooks verify proofs and integrate with ERC-3643 token transfers; attestation logs (EAS) provide audit trails.

**Trust assumptions:**
- Registry operator(s) honesty and availability
- Merkle tree integrity and revocation correctness
- ZK soundness

**Threat model:**
- Registry operator censorship by omission or selective revocation
- Operator compromise reveals membership; mitigated by multi-operator registries
- Registry update latency creates verification windows

**Works best when:**
- Membership set is well-defined (institutional KYC list, DAO whitelist)
- Operator trust is administratively tractable (audited operations, multi-operator)
- Revocation cadence is bounded by governance, not adversarial pace

**Avoid when:**
- Registry operator failure must not break verification (use Issuer-Independent OPRF instead)
- Membership churn is high enough that revocation lag becomes a security problem

### Document ZK Proofs

```yaml
maturity: prototyped
context: both
crops: { cr: medium, o: yes, p: full, s: high }
uses_patterns: [pattern-noir-private-contracts]
example_vendors: []
```

**Summary:** zero-knowledge proofs over cryptographic data from government-issued identity documents (passport NFC, national ID signatures); selectively disclose attributes without revealing the document.

**How it works:** ZKPassport reads the NFC chips of passports, national ID cards, and residence permits (130+ countries per [zkpassport.id](https://zkpassport.id/), October 2026) and proves attributes (age, nationality, sanctions-list non-membership) under Noir / UltraHonk on mobile. Anon Aadhaar verifies UIDAI's RSA signature client-side and emits EVM-verifiable proofs over Indian national ID attributes. The verifier checks the SNARK; no identity data is collected.

**Trust assumptions:**
- Document-issuing authority signature integrity
- Document cryptography (passport ICAO PA, UIDAI RSA) remains unbroken
- Mobile or NFC reader integrity at the client side

**Threat model:**
- Document issuer compromise propagates to all proofs
- Cloned passports / NFC replay; passive authentication alone does not detect a cloned chip, and the anti-cloning checks (Active or Chip Authentication) are optional for issuing states
- Mobile platform compromise reveals derivation material

**Works best when:**
- Attribute proofs are sufficient (sanctions, age, nationality) and full KYC is unnecessary
- User base has NFC-capable devices or supported national IDs
- Issuer cooperation persists for credential refresh

**Avoid when:**
- Issuer cooperation cannot be assumed long-term (use Issuer-Independent OPRF)
- Document cryptography is too weak or restricted for the threat model

### TLS Transcript Proofs

```yaml
maturity: prototyped
context: both
crops: { cr: medium, o: yes, p: partial, s: medium }
uses_patterns: [pattern-zk-tls]
example_vendors: []
```

**Summary:** TLSNotary proves the authenticity of TLS session transcript data from any web2 source (bank portals, government sites, social media); selectively discloses specific fields.

**How it works:** The verifier joins the TLS session with the prover under MPC-TLS, or delegates that role to a Notary that signs commitments to the transcript for reuse with other verifiers. The prover holds the session transcript and discloses selected ranges, optionally with a zero-knowledge proof over redacted data, to attest to specific fields (account balance, government registry record, social signal). The verifier checks the disclosure against the MPC-TLS commitments, or against the Notary's signature in Notary mode.

**Trust assumptions:**
- Notary honesty and availability during the session
- TLS server certificate validity
- Web2 data source not adversarial during the session

**Threat model:**
- Notary compromise corrupts the witnessing function
- TLS server-side data manipulation between sessions
- Disclosure is limited to the transcript ranges the prover committed to ([commit strategy](https://tlsnotary.org/docs/protocol/commit_strategy))

**Works best when:**
- Web2 data source has no API for direct attestation
- Notary trust is administratively manageable (audited, multi-notary, or threshold)
- Freshness expectation is bounded by session timing

**Avoid when:**
- Trusted Notary is incompatible with the threat model
- Many different disclosure patterns are needed and artifact size matters (finer-grained commitments grow storage linearly)

### On-Chain Attestation

```yaml
maturity: production
context: both
crops: { cr: medium, o: yes, p: partial, s: medium }
uses_patterns: [pattern-verifiable-attestation, pattern-zk-wrappers, pattern-erc3643-rwa]
example_vendors: []
```

**Summary:** Trusted issuers publish signed claims on chain (EAS, ONCHAINID via ERC-734/735, W3C VCs); ZK wrappers (OpenAC) add unlinkable presentation across verifiers.

**How it works:** Issuers sign attestations under known keys and publish them on chain. Verifiers reference the attestation and check the issuer signature. With ZK wrappers like OpenAC, the holder presents a proof of attestation possession with selective disclosure, unlinkable across verifiers; no trusted setup, EUDI ARF compatible.

**Trust assumptions:**
- Issuer signing key custody and lifecycle (rotation, revocation)
- Attestation registry availability and correctness
- ZK wrapper soundness (OpenAC) where unlinkability is required

**Threat model:**
- Issuer key compromise enables unauthorized attestations
- Without ZK presentation, verifier-to-verifier linkability via issuer reference
- Attestation revocation must reach all verifiers; staleness creates verification gaps

**Works best when:**
- Permissioned token compliance (ERC-3643), cross-institutional credential sharing
- Issuer governance is mature and audit-ready
- Unlinkability across verifiers is a goal, pair with OpenAC

**Avoid when:**
- Issuer cooperation cannot be assumed long-term (use Issuer-Independent OPRF)
- Verifiers must not be able to correlate via shared issuer references and OpenAC is unavailable

### POD2

```yaml
maturity: prototyped
context: both
crops: { cr: high, o: yes, p: full, s: medium }
uses_patterns: [pattern-private-mtp-auth]
example_vendors: []
```

**Summary:** POD2 (Provable Object Datatype) bundles any data with a cryptographic proof of correctness; event tickets, community badges, poll responses, and access tokens share a composable format.

**How it works:** Each POD is a typed record signed by the issuer; verifiers check the signature and the typed schema. PODs compose: a holder can prove statements over multiple PODs in a single zero-knowledge proof (e.g., "I hold an event POD and a community POD without revealing which event or community").

**Trust assumptions:**
- POD issuer signing keys
- Schema integrity at the ecosystem level
- Composability framework (POD2 library) correctness

**Threat model:**
- Issuer key compromise corrupts all PODs from that issuer
- Schema collision or malleability across communities
- Tooling maturity affects audit confidence

**Works best when:**
- Event gating, community membership, anonymous polls, sybil-resistant access
- Composable credential ecosystems where heterogeneous attestations interoperate
- Community-driven adoption is acceptable

**Avoid when:**
- Institutional-grade audit and assurance is required (community tooling is less mature than EAS / ONCHAINID)

### Issuer-Independent Enrollment via Distributed OPRF

```yaml
maturity: prototyped
context: both
crops: { cr: high, o: yes, p: full, s: high }
uses_patterns: [pattern-private-mtp-auth, pattern-voprf-nullifiers, pattern-regulatory-disclosure-keys-proofs, pattern-verifiable-attestation, pattern-social-recovery]
example_vendors: []
```

**Summary:** Holders prove identity to a threshold vOPRF network, yielding a deterministic enrollment nullifier and a Poseidon leaf in an on-chain LeanIMT; post-enrollment, the on-chain root is the sole trust anchor, no issuer contact during verification.

**How it works:** Any of these credential sources can enroll (passport, national ID, email DKIM, TLS notary, biometric, community vouch). The holder runs RFC 9497 + Jarecki threshold OPRF against the network (TACEO operates a threshold OPRF network, live in production). Output: a deterministic enrollment nullifier and a leaf added to the on-chain LeanIMT. Verification at any verifier checks a zero-knowledge proof against the on-chain root with public inputs (root, scope-bound nullifier, predicate parameters). Sybil resistance is layered: cryptographic (one enrollment per source credential), economic (refundable stake, default 0.1 ETH), social (web-of-trust vouching, future). Recovery via Shamir threshold splitting and guardian-based social recovery.

**Trust assumptions:**
- Threshold OPRF network availability (t-of-n liveness) and fewer than t colluding operators (t colluding operators can bypass the sybil gate and enumerate enrollments)
- Source credential cryptography for enrollment
- On-chain LeanIMT integrity
- Recovery custodian / guardian set independence

**Threat model:**
- OPRF operators accumulate enrollment metadata; mitigated by anonymous communication
- Source-credential compromise enables sybil enrollment from that source; mitigated by economic stake and layered sources
- Predicate parameters are public PoC inputs (production: private predicate circuits)
- Coercion-resistant recovery is an open question

**Works best when:**
- Issuer may become unavailable, hostile, or destroyed
- Sanctions compliance must persist across jurisdictional disruptions
- Multi-source enrollment with issuer-free verification is required

**Avoid when:**
- Single-issuer cooperation is reliable and the operational simplicity of the issuer-dependent approaches is preferred
- Threshold OPRF network operation cannot meet jurisdictional independence requirements

**Implementation notes:** PoC at [Resilient Private Identity](https://github.com/ethsystems/pocs/tree/master/pocs/private-identity/resilient-private-identity) (Noir / UltraHonk, BN254). Production roadmap: BLS12-381 migration via EIP-2537, private predicate circuits, EIP-4337 paymaster or purpose-built relay infrastructure. This sub-approach exposes the `IResilientIdentity` interface used by [Approach: Private Payments, Resilient Disbursement Rails](approach-private-payments.md) and [Approach: Civic Participation](approach-civic-participation.md).

## Comparison

| Axis | Registry Membership | Document ZK | TLS Transcript | On-Chain Attestation | POD2 | Issuer-Independent OPRF |
|---|---|---|---|---|---|---|
| **Maturity** | prototyped | prototyped | prototyped | production | prototyped | prototyped |
| **Context** | i2i | both | both | both | both | both |
| **CROPS** | CR:med O:y P:full S:hi | CR:med O:y P:full S:hi | CR:med O:y P:part S:med | CR:med O:y P:part S:med | CR:hi O:y P:full S:med | CR:hi O:y P:full S:hi |
| **Trust model** | Registry operator | Document issuer | Verifier or Notary + TLS server | Issuer signing key | POD issuer keys | Threshold OPRF + on-chain root |
| **Privacy scope** | k-anonymity in the tree | Attribute-selective | Field-selective within committed transcript ranges | Issuer linkage unless ZK-wrapped | Composable, attestation-based | Multi-source, issuer-independent |
| **Performance** | Low proof cost | Medium (mobile NFC + Noir) | Medium (TLS + ZK) | Low (signature check) | Low | Medium (OPRF + ZK) |
| **Operator req.** | Yes (registry) | No (issuer cooperation only at refresh) | Yes (Notary) | Yes (issuer) | Yes (POD issuer) | Yes (OPRF threshold network) |
| **Cost class** | Low | Medium | Medium | Low | Low | Medium |
| **Regulatory fit** | Strong with view-key scoping | Strong (sanctions, age) | Conditional on Notary governance | Strong (with OpenAC for unlinkability) | Strong for community / event | Strong (issuer-free continuity) |
| **Failure modes** | Registry censor; revocation lag | Issuer key compromise; document clone | Notary compromise; field leakage | Issuer key compromise; staleness | Issuer key compromise; schema collision | OPRF metadata leak; coercion |

## Persona perspectives

### Business perspective

For institutional KYC, sanctions compliance, and ERC-3643 permissioned token transfers, On-Chain Attestation with EAS and ONCHAINID is the production-default; pair with OpenAC for cross-verifier unlinkability where the issuer-linkage signal is competitively sensitive. Registry Membership suits well-defined institutional whitelists where the operator is administratively tractable. Document ZK is suited for sanctions and age verification where attribute-only proofs are sufficient and full KYC is overhead. Issuer-Independent OPRF is the suitable choice when issuer continuity is a documented risk: institutions in sanctioned jurisdictions, humanitarian programs, or governance contexts where the issuer cannot remain available indefinitely.

### Technical perspective

Engineering effort scales by approach. Registry Membership reuses Semaphore-class infrastructure with the lowest proof cost. Document ZK requires mobile NFC integration plus Noir or Circom circuits per document type. TLS Transcript Proofs add MPC-TLS verifier or Notary infrastructure and transcript commitment handling. On-Chain Attestation is the simplest verification surface (signature check) but requires registry hygiene at the issuer. POD2 is composable across heterogeneous attestations but the tooling is community-driven. Issuer-Independent OPRF is the heaviest engineering surface (threshold network, on-chain LeanIMT, recovery infrastructure) but is the only category designed to survive single-issuer failure.

### Legal & risk perspective

This is a perspective for legal review by the deploying institution, not legal advice. Each approach exposes a distinct disclosure interface.

Registry Membership pairs scoped view keys with EAS-logged audit trails; the registry operator would typically be the named regulated party. Document zero-knowledge proofs disclose only the attribute predicate's public output; jurisdictional acceptance varies (ZKPassport was used at the Aztec sale to attest non-sanctioned status across OFAC / Swiss / EU / UK lists, which is a deployment data point, not a regulatory endorsement). TLS Transcript Proofs raise classification questions about the Notary's role.

On-Chain Attestation aligns with existing identity-issuer registry conventions; whether a given regulator accepts EAS or ONCHAINID-style attestations as compliance evidence is a per-jurisdiction question, and OpenAC layers unlinkability without breaking the audit trail at the issuer. POD2 is community-grade and may not match institutional-grade evidence requirements. Issuer-Independent OPRF would require legal sign-off on the threshold-network composition, the recovery model, and the coercion-resistance posture; the document does not claim sufficiency for any specific regime.

## Recommendation

### Default

For institutional permissioned-asset access today, default to On-Chain Attestation with EAS and ONCHAINID; add OpenAC when cross-verifier unlinkability matters. For institutional KYC whitelists with operator-controlled membership, default to Registry Membership Proofs (Semaphore-class). For sanctions and age verification on a global user base, default to Document ZK (ZKPassport) on supported documents.

### Decision factors

- If issuer continuity is a documented risk (sanctions, jurisdictional disruption, humanitarian context), choose Issuer-Independent OPRF and design the threshold network for jurisdictional diversity.
- If web2 data sources without APIs must be brought on-chain, choose TLS Transcript Proofs and validate the Notary governance.
- If the ecosystem requires composable attestations across heterogeneous communities, choose POD2 and accept the community-tooling maturity trade-off.

### Hybrid

Run On-Chain Attestation as the default for production permissioned flows; layer Document ZK for sanctions screening at the entry boundary; enroll high-risk users (institutions in jurisdictionally exposed locations, humanitarian recipients) into the Issuer-Independent OPRF as a continuity backup. Recovery uses Shamir + guardian quorum across the same enrollment. Coercion-resistance posture is documented per program; production-grade anti-coercion mechanism is an open research item.

## Open questions

1. **Multi-Address Efficiency.** Proving ownership of multiple EOAs from the same seed without revealing derivation patterns.
2. **Regulatory Recognition.** Legal recognition of zero-knowledge proofs as sufficient compliance evidence per jurisdiction.
3. **Notary Trust.** Guaranteeing Notary trustworthiness in zk-TLS deployments; threshold or audited Notary networks.
4. **Cross-Credential Composability.** Combining proofs from heterogeneous sources (passport + KYC attestation + community POD) in a single verification.
5. **Credential Revocation.** Revocation across heterogeneous issuers without a central authority.
6. **Guardian Recovery Design.** Minimum guardian set for anti-coercion; acceptable coordination overhead for quorum recovery.
7. **Attribute Freshness.** Without an issuer to refresh, how stale attributes (expired passports, changed nationality) are handled in OPRF-anchored enrollment.
