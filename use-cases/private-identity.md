---
title: Private Identity
status: ready
primary_domain: Identity & Compliance
secondary_domain: Civic Coordination & Governance
---

## 1) Use Case

Prove identity claims, membership, eligibility, or attribute possession on-chain without revealing the underlying identity or creating linkable activity. Authentication is one key application; others include credential portability, sybil resistance, and selective disclosure. Applies to financial compliance (KYC registry membership), governance (anonymous voting), national identity (selective disclosure from government documents), and community membership (sybil-resistant access).

## 2) Additional Context


**Validated deployments:**

- **National identity:** ZKPassport used in Aztec token sale (120+ country passports, NFC + Noir circuits); Anon Aadhaar
- **Governance:** NounsDAO Private Voting (Aztec + Noir storage proofs); MACI deployed at ETHDam, ETHMexico, ETH Tegucigalpa (anti-collusion)
- **Community/sybil:** [POD2](https://github.com/0xPARC/pod2) (0xPARC); World ID (18M+ verified humans)
- **Email identity:** zkEmail (DKIM proofs, email wallets on Arbitrum)

## 3) Actors

Credential Issuer (bank, government, DAO, university) · Prover / Subject (investor, citizen, voter, community member) · Verifier (smart contract, institution, election system) · Registry Operator (maintains on-chain commitment: Merkle root, attestation registry, membership set) · Auditor / Regulator (entity with scoped disclosure rights: financial regulator, election observer) · Wallet / Proof Agent (client-side proof generation: mobile wallet, browser extension, NFC reader)

## 4) Problems

### Problem 1: Authentication Without Identity Leakage

Current authentication methods (e.g., message signatures) prove key control but expose addresses and create linkability between provers and verifiers. Two distinct but related needs emerge:

1. **Authentication**: prover demonstrates membership or attribute possession without revealing identity.
2. **Interaction privacy**: prevent on-chain linkability between prover addresses and verifier contracts.

**Requirements:**

- **Must hide:** prover identities, links between verifiers and provers, links between prover EOAs
- **Public OK:** registry roots, verifier contract addresses, [compliance attestations](../patterns/pattern-verifiable-attestation.md)
- **Auditor access:** scoped access to Merkle inclusion proofs, registry updates, or decryption keys where required
- **Settlement:** proof verification + transaction execution
- **Ops:** resilience against replay attacks; low-cost proof generation; interop across rollups; proof generation on consumer hardware

### Problem 2: Credential Source Diversity

Different domains require different credential sources: government-issued IDs (passports, national IDs), institutional attestations (KYC providers), biometric enrollment, email ownership (DKIM), event attendance, and on-chain history. No single canonical identity system covers all use cases.

**Requirements:**

- Must support: document-based (passport NFC), attestation-based (EAS/VC), biometric-enrollment-based, email-based (DKIM), membership-based (Merkle tree), TLS-based (web2 data export)
- Must not require: a single canonical identity system or global registry
- Interop: credentials from different sources should be composable (e.g., prove KYC + jurisdiction + accreditation from different issuers in one proof)

### Problem 3: Sybil Resistance Without Surveillance

Systems that distribute value (governance votes, token distributions) must prevent double-claiming without building a surveillance database. This requires deterministic, scope-bound nullifiers that prevent repeat actions without linking to the underlying identity.

**Requirements:**

- Must prevent: double-claiming, double-voting, distribution farming
- Must preserve: unlinkability across different scopes/services
- Must handle: revocation of compromised credentials without re-identifying holders
- Reference: [vOPRF Nullifiers pattern](../patterns/pattern-voprf-nullifiers.md) for nullifier generation primitives

### Problem 4: Credential Portability & Reuse

Institutions repeatedly collect the same credentials (KYC/KYB) from counterparties during onboarding. Reusable, privacy-preserving credentials that work across institutional contexts reduce onboarding cost and friction. Key challenges include cross-border recognition (a credential issued in one jurisdiction accepted in another), revocation propagation without de-anonymization (revoking a compromised credential without revealing which holder was affected), and credential freshness guarantees (proving a credential is still valid, not just that it was once issued).

**Requirements:**

- Must support: reuse of a single credential across multiple relying parties without re-disclosure of underlying data
- Must not require: a centralized credential database or single point of trust
- Must handle: cross-border credential recognition (e.g., eIDAS mutual recognition), revocation propagation, and credential expiry/refresh
- Reference: [Verifiable Attestation pattern](../patterns/pattern-verifiable-attestation.md) for attestation lifecycle primitives; [OpenAC](https://eprint.iacr.org/2026/251) for privacy-preserving access control across relying parties

## 5) Recommended Approaches

| Credential Source | Primary Approach | Example Deployments |
| --- | --- | --- |
| Institutional KYC registry | Merkle tree membership proofs | ERC-3643 issuances, Semaphore |
| Government ID (passport) | Document ZK proofs (NFC + Noir) | ZKPassport / Aztec token sale |
| Government ID (national) | Document RSA signature + ZK | Anon Aadhaar / ETHIndia 2024 voting |
| Email | DKIM signature proofs | zkEmail email wallet (Arbitrum trial, now Base Sepolia) |
| Web2 data source | TLS transcript proofs | TLSNotary |
| Biometric enrollment | Iris/face hash + membership proof | World ID |
| Event/community | POD2 | [POD2](https://github.com/0xPARC/pod2) (0xPARC) |
| Multi-party private inputs | Collaborative proving (co-SNARK) | TACEO |
| On-chain attestation | EAS / ONCHAINID / W3C VC | Tokeny, EAS |
| Civil registry / vital records | Government registry attestation + ZK | eIDAS 2.0, EUDI pilots |
| Professional license / certificate | Issuer attestation + revocation registry | EBSI, national licensing boards |

See detailed architecture and trade-offs in [**Approach: Private Identity**](../approaches/approach-private-identity.md).

## 6) Open Questions

1. How practical is it to prove ownership of multiple EOAs derived from the same seed without revealing derivation patterns?
2. Should multi-ownership proofs be handled at the wallet layer (BIP-32 style derivations) or at the protocol layer (aggregated ZK proofs)?
3. How to establish cross-credential interoperability, e.g., combine a ZKPassport proof with an institutional attestation in a single transaction?
4. What trust models are acceptable for biometric enrollment systems in different regulatory contexts?
5. How to handle credential revocation across heterogeneous issuers without a central revocation authority?
6. How do these building blocks map onto existing and emerging compliance frameworks (EUDI ARF, eIDAS 2.0, MiCA) across jurisdictions?
7. How do government-issued verifiable credentials (eIDAS 2.0, EUDI Wallet) integrate with on-chain attestation registries for institutional KYC?
8. What models enable credential reuse across institutions without creating a centralized identity database?

## 7) Notes And Links

- **Standards:** [ERC-3643 ONCHAINID](https://www.erc3643.org/), [EAS](https://attest.org/), [W3C Verifiable Credentials 2.0](https://www.w3.org/TR/vc-data-model-2.0/), [EIP-5564](https://eips.ethereum.org/EIPS/eip-5564)
- **ZK Frameworks:** [Semaphore](https://semaphore.pse.dev/), [Noir/Barretenberg](https://docs.aztec.network/), [Circom/Groth16](https://docs.circom.io/)
- **Credential Systems:** [ZKPassport](https://zkpassport.id/), [Self](https://self.xyz/), [Rarimo](https://rarimo.com/), [Anon Aadhaar](https://github.com/anon-aadhaar), [zkEmail](https://zk.email/), [TLSNotary](https://tlsnotary.org/), [POD2](https://github.com/0xPARC/pod2), [OpenAC](https://eprint.iacr.org/2026/251)
- **Validated Deployments:** [WFP Building Blocks](https://www.wfp.org/building-blocks), [OpenCerts](https://www.opencerts.io/)
- See also: [EPIC map](https://epic-webapp.vercel.app/) (GovTech & EPIC team, demo data): verifiable credentials, KYC/KYB, licensing, selective disclosure
- **Related Patterns:** [Private MTP Auth](../patterns/pattern-private-mtp-auth.md), [zk-TLS](../patterns/pattern-zk-tls.md), [Verifiable Attestation](../patterns/pattern-verifiable-attestation.md), [vOPRF Nullifiers](../patterns/pattern-voprf-nullifiers.md), [Selective Disclosure](../patterns/pattern-regulatory-disclosure-keys-proofs.md), [co-SNARK](../patterns/pattern-co-snark.md)
