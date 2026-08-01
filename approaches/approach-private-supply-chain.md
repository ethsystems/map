---
title: "Approach: Private Supply Chain"
status: ready
last_reviewed: 2026-08-02

use_case: private-supply-chain
related_use_cases: [private-registry, private-identity]

primary_patterns:
  - pattern-verifiable-attestation
  - pattern-private-geospatial-attestation
  - pattern-l2-encrypted-offchain-audit
supporting_patterns:
  - pattern-commit-and-prove
  - pattern-regulatory-disclosure-keys-proofs
  - pattern-zk-wrappers

open_source_implementations:
  - url: https://github.com/ethereum-attestation-service/eas-contracts
    description: "Ethereum Attestation Service contracts (production)"
    language: Solidity
---

# Approach: Private Supply Chain

## Problem framing

### Scenario

A commodity moves from producer to EU importer through a cooperative, an exporter, a shipper, and a customs broker. Each handoff produces a compliance-relevant fact. The importer must show a regulator that the goods are authentic, legally produced, and not linked to deforested land. None of the parties will publish supplier identities, volumes, or routing, because that data reconstructs sourcing strategy and margin structure for any competitor watching the chain.

From 30 December 2026 the EU Deforestation Regulation adds a sharper requirement: plot-level geolocation at six decimal digits, as polygons above four hectares, tied to specific products. Pharmaceutical serialization under FMD and DSCSA already imposes unit-level tracking on a different set of goods. Both regimes demand granular data that is commercially sensitive, and in the smallholder case personal.

### Requirements

- Prove product authenticity and regulatory compliance to a verifier who sees no upstream detail.
- Keep supplier identities, prices, volumes, and routing confidential between adjacent parties.
- Give regulators a path to full provenance for investigations and recalls.
- Verify at point of use, including customs checkpoints with poor connectivity.
- Interoperate with existing ERP and serialization systems.

### Constraints

- Parties span jurisdictions with conflicting disclosure and data protection rules.
- Customs clearance needs near real-time verification.
- Volume reaches millions of units per manufacturer per year.
- Smallholders lack infrastructure to run proving or key management themselves.

## Approaches

### Attestation Chain With Selective Disclosure

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: yes, p: partial, s: high }
uses_patterns: [pattern-verifiable-attestation, pattern-regulatory-disclosure-keys-proofs]
```

**Summary:** Each party signs an attestation for its own step. Downstream parties verify the chain's shape and endpoints without reading intermediate content.

**How it works:** Every handoff produces a signed attestation referencing the previous one by hash. Content is encrypted to the parties entitled to read it, with a viewing key escrowed for regulators. A verifier checks signatures and linkage to confirm an unbroken chain from a recognised origin.

**Trust assumptions:** Issuers sign honestly. Revocation registries stay queryable. The escrow holder does not leak viewing keys.

**Threat model:** A colluding pair can forge a link for goods that never moved. The chain proves attestation continuity, not physical custody.

**Works best when:** Parties are regulated entities with legal recourse against each other.

**Avoid when:** Participants include producers who cannot manage signing keys.

### Zero-Knowledge Geospatial And Compliance Proofs

```yaml
maturity: concept
context: both
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-private-geospatial-attestation, pattern-commit-and-prove]
```

**Summary:** Prove the regulatory predicate rather than disclosing the data behind it. A plot polygon is proven not to intersect a committed deforestation layer, and the coordinates never leave the operator.

**How it works:** Producers commit to plot geometry once. A published reference layer is committed and versioned against a baseline date. A circuit proves non-intersection, and the proof is bound to a due diligence statement reference through an attestation wrapper.

**Trust assumptions:** The published reference layer is accurate for the stated baseline. The prover runs the circuit faithfully.

**Threat model:** Whoever publishes the layer defines ground truth, so a stale or manipulated layer produces sound proofs of false facts. Small producer populations remain re-identifiable from predicate results plus shipment metadata.

**Works best when:** A regulator has named the geographic test and a public layer already encodes it.

**Avoid when:** The reference layer is disputed, or the verifier needs coordinates for physical inspection.

### Encrypted Off-Chain Audit With On-Chain Anchoring

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns: [pattern-l2-encrypted-offchain-audit, pattern-commit-and-prove]
```

**Summary:** Keep batch and logistics records off-chain and encrypted, anchoring commitments on-chain so that later disclosure is provably untampered.

**How it works:** Operators write serialization and handling records to an encrypted store, publishing periodic commitments. Authorised parties receive decryption capabilities scoped to a batch or time window. Any disclosed record is checked against its anchor.

**Trust assumptions:** The storage operator preserves availability. Key distribution reaches the right parties.

**Threat model:** An operator who withholds data blocks audit without breaking any commitment. Anchoring proves integrity, not availability.

**Works best when:** Volume is high and most records are never read.

**Avoid when:** Verifiers need to check records without a prior relationship to the operator.

## Comparison

| | Attestation Chain | ZK Geospatial Proofs | Encrypted Audit + Anchoring |
| --- | --- | --- | --- |
| **Maturity** | documented | concept | prototyped |
| **Context** | i2i | both | i2i |
| **CROPS** | cr medium, o yes, p partial, s high | cr medium, o partial, p partial, s medium | cr medium, o partial, p full, s medium |
| **Trust model** | Trusted issuers, escrowed viewing keys | Trusted layer publisher, trustless verification | Trusted storage operator |
| **Privacy scope** | Hides step content, reveals chain shape | Hides source data, reveals predicate result | Hides records fully until disclosure |
| **Performance** | Signature checks, fast | Proving cost scales with polygon vertices | Write-heavy, cheap verification |
| **Operator req.** | Key management per party | Proving service, layer distribution | Storage and key distribution |
| **Cost class** | Low | High per proof | Low per record, moderate anchoring |
| **Regulatory fit** | Strong for serialization mandates | Strong for EUDR geolocation | Strong for recall and audit trails |
| **Failure modes** | Colluding forgery, key loss | Stale layer, re-identification | Operator withholding, key distribution gaps |

## Persona perspectives

### Business perspective

The commercial risk in supply chain transparency is not regulatory exposure but competitive exposure. A published provenance trail lets a competitor reconstruct supplier dependencies, seasonal volumes, and margin structure from public data, which is a permanent disadvantage traded for a one-off compliance win. That calculus explains why operators have preferred closed platforms even where shared infrastructure would cost less. The attestation chain preserves existing commercial relationships and needs the least change to how parties already work, which makes it the realistic entry point. Zero-knowledge geospatial proving carries real cost per shipment and buys something specific in return, which is the ability to satisfy EUDR without handing plot coordinates to every downstream buyer. Where a business already holds the coordinates and treats them as a sourcing asset, that trade is straightforward. Where it does not, the proving obligation lands on cooperatives least able to absorb it.

### Technical perspective

The binding problem dominates. Every architecture here proves something about records, and none proves that records correspond to physical goods. Attestation chains prove continuity of signatures. Geospatial circuits prove a polygon satisfies a predicate. Anchoring proves a disclosed record matches what was committed. The gap between those proofs and reality is closed by attestation from a party with physical presence, which means issuer selection remains the security-critical decision regardless of the cryptography layered above it. Two implementation details cause most failures in practice. Hash function mismatch between on-chain digests and in-circuit field-friendly hashes breaks commitment parity in ways that surface late. Reference layer versioning creates a re-proving obligation across every affected plot each time the layer updates, which needs planning before the volume arrives rather than after.

### Legal & risk perspective

Several questions here are unresolved rather than merely difficult. Plot polygons for smallholders function as personal data under GDPR while also being commercial data belonging to a buyer, and it is unsettled whether a commitment to a polygon constitutes pseudonymised or anonymised processing under the EDPB's current guidelines. That classification determines whether erasure rights attach to an on-chain anchor. Regulators have not stated whether a zero-knowledge predicate proof discharges an Article 9 information requirement that is written in terms of collecting and holding coordinates, and the plain reading suggests collection remains mandatory even where onward disclosure is minimised. Cross-border disclosure adds another layer, because a viewing key held in one jurisdiction may be reachable by process in another. Jurisdictional coverage in this map that bears on these questions sits in the EUDR and EU data protection cards.

## Recommendation

### Default

Start with the attestation chain, because it composes with existing serialization systems and needs no new cryptographic infrastructure. Layer geospatial proving on top where a geographic mandate applies, rather than treating it as an alternative. The two solve different problems, and the second depends on the first to bind proofs to real parties.

### Decision factors

- If the driver is EUDR, geospatial proving is on the critical path and the reference layer question needs answering before anything is built.
- If the driver is pharmaceutical serialization, the attestation chain plus anchoring covers the mandate, and geospatial proving is out of scope.
- If producers cannot hold keys, plan for a cooperative or buyer to act as prover, and record that this reintroduces the intermediary the design was meant to remove.
- If verification happens offline at customs, distribution of layer commitments becomes an operational requirement rather than a detail.

### Hybrid

The realistic production shape combines all three. Attestations bind parties and steps. Geospatial proofs discharge the geographic predicate without disclosure. Encrypted audit with anchoring holds the volume of routine records that no one reads until a recall or investigation makes them matter.

## Open questions

1. Does a zero-knowledge predicate proof satisfy an information requirement drafted in terms of collecting coordinates, or does collection remain mandatory with only onward disclosure minimised?
2. Who publishes and commits the reference deforestation layer, and what recourse exists when that layer is wrong?
3. Is a commitment to a plot polygon pseudonymised or anonymised processing under GDPR, and do erasure rights attach to the anchor?
4. What is the minimum viable attestation schema that customs authorities in different jurisdictions would each accept without bilateral negotiation?
5. How should re-proving obligations be handled when a reference layer updates across millions of anchored plots?
6. Can verification at low-connectivity checkpoints rely on cached layer commitments without opening a staleness attack?
