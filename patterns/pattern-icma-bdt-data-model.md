---
title: "Pattern: ICMA Bond Data Taxonomy (canonical bond terms)"
status: ready
maturity: production
type: standard
layer: offchain
last_reviewed: 2026-10-13

works-best-when:
  - You want machine-readable bond terms across tools.
  - Multiple issuance platforms need to interoperate without bespoke mappings.
avoid-when:
  - A jurisdiction or domain mandates a different canonical schema.

context: i2i

crops_profile:
  cr: none
  o: yes
  p: none
  s: high

crops_context:
  cr: "Schema governance sits with a single standards body (ICMA). Could reach `medium` if the schema is published as a permissionless open registry with attestation-anchored contributions and no approval gate."
  o: "Open specification, publicly documented. Implementations can reuse the schema under ICMA's royalty-free license, though ICMA retains the IP and schema evolution is gated by the standards body."
  p: "Schema itself contains no participant data; only field definitions. Raw bond data stays off-chain, with only hashes anchored on-chain, so the pattern does not expose confidential details."
  s: "Rides on the correctness of off-chain validators and the integrity of hash anchoring. Well-defined schemas reduce integration errors and make regulator reconciliation straightforward."

post_quantum:
  risk: low
  vector: "Hash-based anchoring is not directly affected. Any ECDSA-signed off-chain attestations over BDT records are vulnerable to a CRQC, but the canonical schema itself has no cryptographic dependency."
  mitigation: "Use post-quantum signatures for attestations over BDT records; hash-based commitments for on-chain anchoring remain safe."

standards: [ICMA-BDT]

related_patterns:
  composes_with: [pattern-crypto-registry-bridge-ewpg-eas, pattern-verifiable-attestation, pattern-regulatory-disclosure-keys-proofs]
  see_also: [pattern-private-iso20022]
---

## Intent

Use the ICMA Bond Data Taxonomy as the canonical schema for bond terms. A shared data model avoids fragmentation across issuance platforms, registrars, and regulators, and provides a stable foundation for attestations and zero-knowledge proofs over bond data.

## Components

- Canonical schema (XML Schema, with a JSON format also available) defines the key terms of a bond term sheet: amounts, currency, dates, interest terms, parties, governing law, selling restrictions, and DLT platform fields. It does not define lifecycle event messages (coupon payments, redemptions); event records can reference BDT fields but need a separate schema.
- Schema validator checks that bond records conform to the canonical definitions before they are signed, hashed, or anchored.
- Compression layer converts verbose representations (for example XML) into binary or compact serialization for efficient on-chain anchoring.
- Registrar integration maps internal registrar records to the canonical schema so existing systems can emit conformant data without rewriting their back office.
- Attestation anchor posts hashes of conformant records to an on-chain attestation registry so any verifier can check that a disclosed document matches what the registrar recorded.

## Protocol

1. [operator] Author bond terms in the canonical schema.
2. [operator] Validate the record against the schema and compress for storage.
3. [operator] Store the record (on-chain hash anchor, off-chain canonical payload).
4. [auditor] Use the same schema when preparing selective-disclosure proofs or attestations.
5. [regulator] Verify consistency between disclosed payloads and on-chain hash anchors.

## Guarantees & threat model

Guarantees:

- Interoperable, regulator-friendly bond data across platforms.
- Easier proofs and attestations over common fields without bespoke per-issuer schemas.
- Clean baseline for composing with hash-anchored registries and zero-knowledge disclosure patterns.

Threat model:

- Schema governance integrity. If the standards body publishes conflicting revisions, validators may diverge on what counts as conformant.
- Registrar mapping correctness. A mismapping at integration time can silently break consistency between internal records and canonical output.
- Out of scope: data confidentiality. The schema is a structural artifact, not a privacy tool; confidentiality must come from accompanying patterns (hash anchoring, selective disclosure, ZK proofs).

## Trade-offs

- Up-front mapping effort is required to wire existing registrar systems into the canonical schema.
- Schema extensions for new product types (complex derivatives, non-standard covenants) need standards-body involvement, which can lag market innovation.
- Not a privacy mechanism on its own. Must be paired with hashing, attestation, or zero-knowledge patterns to protect the underlying data.

## Example

An issuance platform publishes bond terms in the canonical schema. The registrar validates the record and anchors its hash via an on-chain attestation registry. A regulator later receives the full payload off-chain and verifies the hash matches the on-chain anchor, confirming the record has not been altered since issuance.

## See also

- [ICMA Bond Data Taxonomy overview](https://www.icmagroup.org/fintech-and-digitalisation/fintech-advisory-committee-and-related-groups/bond-data-taxonomy/)
- [ICMA BDT FAQs, April 2026 (PDF)](https://www.icmagroup.org/assets/documents/Regulatory/FinTech/Bond-data-taxonomy/ICMA-Bond-Data-Taxonomy-FAQs-April-2026-280426.pdf)
- [Use case: Private Bonds](../use-cases/private-bonds.md) · [Approach: Private Bonds](../approaches/approach-private-bonds.md)
