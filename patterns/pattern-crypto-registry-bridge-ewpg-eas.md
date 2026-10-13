---
title: "Pattern: Crypto-registry bridge (eWpG) with EAS mirroring"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-10-13

works-best-when:
  - Germany or eWpG applies and a licensed crypto-register is required.
avoid-when:
  - You can legally replace the register on day one.

context: i2i

crops_profile:
  cr: none
  o: partial
  p: full
  s: medium

crops_context:
  cr: "The registrar is the sole gatekeeper and can refuse or delay registrations. Reaches `medium` if an open API standard enables alternative registrar implementations, reducing single-registrar lock-in."
  o: "Middleware is often proprietary. Improves to `yes` by publishing the registrar integration layer and EAS schema definitions under an open-source license."
  p: "No PII is posted on-chain; only hashes and attestations. Plaintext legal records stay with the licensed registrar and are disclosed off-chain under NDA."
  s: "Rides on the registrar's operational integrity and the EAS attester's key management. Improves to `high` by requiring multiple independent EAS attesters to cross-validate registrar claims."

post_quantum:
  risk: medium
  vector: "Attestation signatures rely on standard ECDSA and inherit PQ exposure. Hashes anchored on-chain are PQ-safe."
  mitigation: "Migrate attestation signatures to PQ-safe schemes as ecosystem standards mature. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [EAS]

related_patterns:
  composes_with: [pattern-regulatory-disclosure-keys-proofs, pattern-icma-bdt-data-model, pattern-verifiable-attestation]
  see_also: [pattern-erc3643-rwa]

open_source_implementations: []
---

## Intent

Operate with a licensed crypto-register today while mirroring key legal facts (issuance, transfers, liens) as on-chain attestations. The registrar remains the legal source of truth; the on-chain anchor provides tamper evidence and paves the way for later automation once regulation allows.

## Components

- Licensed crypto-securities registrar under eWpG keeps the register on a decentralised, tamper-protected recording system (eWpG §4(11), §16) and holds holder PII, the deposited issuance terms (§5), and signed records off-chain. The register is the sole source of legal truth.
- Registrar API integration layer exposes signed records for each legal event (issuance, transfer, lien) to downstream systems.
- On-chain attestation schema defines the structure of mirrored facts: event type, record hash, registrar identifier, timestamp. PII never appears on-chain.
- Middleware attester reads the registrar's signed records, computes the hash, and posts an attestation on-chain under the schema.
- Reconciliation and audit tooling matches the on-chain anchor to the off-chain registrar record and flags drift.

## Protocol

1. [operator] Registrar records the legal event in the off-chain register and emits a signed record containing the event details and a record hash.
2. [operator] Middleware consumes the signed record, verifies the registrar's signature, and constructs an attestation containing the record hash, event type, and timestamp.
3. [contract] The attestation is posted on-chain under the agreed EAS schema. No PII is included.
4. [auditor] Later audits match the on-chain anchor to the registrar record: retrieve the registrar's plaintext record under NDA, recompute the hash, and confirm it equals the anchored attestation.
5. [auditor] Discrepancies trigger the incident runbook: the registrar remains legally authoritative, but a mismatch is evidence of tampering in one of the two systems.

## Guarantees & threat model

Guarantees:

- Legal compliance with eWpG is preserved; the licensed registrar remains the authoritative record keeper.
- On-chain anchor provides tamper-evident linkage between the registrar's private ledger and a public timestamp.
- Anyone can verify that an attestation exists, who signed it, and when, without the plaintext; matching it to a record needs the plaintext.

Threat model:

- Trust in the registrar as sole gatekeeper. The registrar can refuse or delay any registration with no on-chain bypass.
- Middleware attester key compromise allows forged anchors. Multi-attester cross-validation mitigates this.
- Collusion between the registrar and the middleware can produce consistent but fraudulent records on both sides. Independent attesters reduce but do not eliminate this risk.
- PII exposure is out of scope for the on-chain layer by design; the registrar's access controls govern all PII handling.

## Trade-offs

- Two sources of truth require ongoing reconciliation. Drift between the registrar and the on-chain anchor must be detected and resolved operationally.
- Strong incident and runbook discipline is required. A mismatch is not self-resolving; it triggers a manual investigation.
- Registrar dependency limits availability. Downtime at the registrar halts new anchors even if the on-chain side is healthy.
- The pattern is explicitly transitional. eWpG already requires the crypto-securities register to run on a decentralised recording system, but under a licensed registrar; the pattern assumes a later setup where register entries live directly on the public chain and automation can replace the middleware.

## Example

- An issuer records a bond issuance with the licensed registrar under eWpG.
- The registrar emits a signed record.
- The middleware hashes the record and posts an EAS attestation on Ethereum.
- A year later, an auditor retrieves the registrar's plaintext record under NDA, recomputes the hash, and confirms it matches the on-chain attestation, establishing integrity.

## See also

- [eWpG statute text (German)](https://www.gesetze-im-internet.de/ewpg/)
- [BaFin crypto-securities registration](https://www.bafin.de/EN/unternehmen-maerkte/erlaubnis-registrierung/geschaefte-krypto/kryptowertpapierregisterfuehrung/kryptowertpapierregisterfuehrung_node_en.html)
- [EAS documentation](https://docs.attest.org/)
- [Use case: Private Bonds](../use-cases/private-bonds.md) · [Approach: Private Bonds](../approaches/approach-private-bonds.md) · [Jurisdiction: DE eWpG](../jurisdictions/de-eWpG.md)
