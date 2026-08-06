---
title: "Domain: Identity & Compliance"
status: draft
description: "Prove eligibility and authority without exposing PII, even when the issuer is unavailable or adversarial."
---

## TLDR
- Institutional credentials: KYC/AML, accreditation, allow/deny, attestations, revocation, scoped regulator disclosure.
- Public-sector eligibility and issuer-independent verification: digital ID wallets, residency, age, citizenship, and benefit-eligibility claims that verify without contacting the issuer.
- Resilience identity: prove attributes after the issuer goes offline, refuses re-issuance, mass-revokes, or turns adversarial; social or web-of-trust recovery without re-exposing PII.
- Across all three: public verification without publishing PII; unlinkable proofs across contexts; scoped disclosure for legitimate oversight.

## Primary use cases
- [Private Identity](../use-cases/private-identity.md)
- [Resilient Identity Continuity](../use-cases/resilient-identity-continuity.md)
- (Cross-cut; applied in all domains as a prerequisite)

## Approaches
- [Approach: Private Identity](../approaches/approach-private-identity.md)

## Shortest-path patterns
- [Private MTP Authentication](../patterns/pattern-private-mtp-auth.md)
- [zk-KYC/ML + ONCHAINID (ERC-734-735)](../patterns/pattern-zk-kyc-ml-id-erc734-735.md)
- [Selective disclosure (view keys + proofs)](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Verifiable Attestation](../patterns/pattern-verifiable-attestation.md)
- [vOPRF Nullifiers](../patterns/pattern-voprf-nullifiers.md)
- [Crypto-registry bridge (eWpG) + EAS](../patterns/pattern-crypto-registry-bridge-ewpg-eas.md)

## Adjacent vendors
- [Privacy Pools](../vendors/privacypools.md)
- [Curvy](../vendors/curvy.md)
- [Peer](../vendors/peer.md)
- [EY Nightfall](../vendors/ey.md)
## Reference
- [Privacy Standards Survey](../surveys/privacy-standards.md) — catalog of institutional privacy standards (ERC-3643 through ERC-8065), with gap analysis and decision guidance.
