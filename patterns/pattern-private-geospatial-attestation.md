---
title: "Pattern: Private Geospatial Attestation"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-08-02

works-best-when:
  - A regulator needs a geographic predicate proven, not the underlying coordinates.
  - Plot boundaries reveal sourcing relationships that competitors would exploit.
  - A public, versioned reference layer already defines the thing being tested against.
avoid-when:
  - The reference layer is disputed or unpublished, leaving nothing sound to commit to.
  - Coordinates are already public, where a plain attestation is cheaper.
  - The verifier needs the coordinates themselves, such as for physical inspection.

context: both
context_differentiation:
  i2i: "Operator to competent authority, where both sides are regulated entities with legal recourse. The confidentiality at stake is commercial, because plot polygons expose sourcing networks and volumes to competitors. Either side can escalate through established administrative channels, so the privacy requirement is about trade secrecy rather than protection from the counterparty."
  i2u: "Smallholder to buyer or authority, where the producer's plot polygon is simultaneously their livelihood and their personal data, and they cannot refuse to disclose it and still reach the market. The producer has no negotiating power over how the coordinates are stored or reused once handed over. CROPS must protect the producer here, which means disclosure has to be predicate-shaped by default rather than by the buyer's goodwill."

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "A producer who cannot obtain a proof cannot sell into the regulated market, so the prover and the layer publisher are both chokepoints. CR improves where multiple provers can serve the same committed layer, and degrades sharply where one platform holds both the layer and the proving service."
  o: "Reference layers such as the JRC global forest cover maps are published openly, and GeoJSON is an open format. The proving circuits for polygon geometry are not yet standardised or widely implemented, which is what holds this at `partial`."
  p: "Hides the polygon and its coordinates while revealing the predicate result and which layer version it was tested against. The plot's existence, its approximate commodity, and the fact that it passed remain visible. For a producer whose farm is one of a handful in a region, the predicate plus commodity can still narrow identity considerably."
  s: "Rests on the correctness of the geometric circuit and on the integrity of the committed layer. A stale or manipulated layer produces proofs that are cryptographically sound and factually wrong."

post_quantum:
  risk: medium
  vector: "Proofs are ephemeral compliance artefacts rather than long-lived secrets, so harvest-now-decrypt-later pressure is low. A CRQC breaking the proof system would allow forged non-intersection proofs for plots that were in fact deforested."
  mitigation: "Hash-based commitments for the layer and the plot, with a hash-based proof system for the geometry. See [Post-Quantum Threats](../domains/post-quantum.md)."

visibility:
  counterparty: [predicate result, layer version]
  chain: [plot commitment, layer commitment, proof]
  regulator: [full coordinates under viewing key or legal process]
  public: [that some committed plot passed against a named layer version]

standards: [RFC-7946, EAS]

related_patterns:
  composes_with: [pattern-verifiable-attestation, pattern-commit-and-prove]
  see_also: [pattern-regulatory-disclosure-keys-proofs, pattern-l2-encrypted-offchain-audit]

open_source_implementations: []
---

## Intent

Prove that a plot of land satisfies a geographic condition, such as not intersecting a deforested region, without revealing where the plot is.

## Components

- Plot geometry: a polygon at regulatory precision, held by the producer or operator and never published.
- Plot commitment: a binding commitment to that geometry, anchored on-chain so the same plot can be re-proved later without re-disclosure.
- Committed reference layer: a published deforestation or land-cover layer, reduced to a commitment and versioned against a stated baseline date. This is the load-bearing component.
- Geometric circuit: proves the polygon lies wholly outside the layer's flagged cells.
- Attestation wrapper: binds the proof to an issuer and to a compliance record such as a due diligence statement reference.
- Disclosure path: a viewing key or legal process that opens the coordinates to an authority during an investigation.

## Protocol

1. [issuer] Publish a versioned commitment to the reference layer for a stated baseline date.
2. [operator] Collect plot geometry from the producer at the precision the regime requires.
3. [operator] Commit to the polygon and anchor the commitment, keeping coordinates off-chain.
4. [prover] Generate a proof that the committed polygon does not intersect the flagged region of the committed layer version.
5. [contract] Verify the proof against the layer commitment and the plot commitment, and record the outcome.
6. [auditor] Verify independently by re-running the circuit against the same published layer version.
7. [regulator] Obtain coordinates through the disclosure path when an investigation requires them.

## Guarantees & threat model

- Hides plot coordinates and boundary shape from buyers, competitors, and downstream verifiers.
- Proves one geometric predicate against one named layer version. It does not prove legality in general.
- Threat model: whoever publishes the reference layer defines ground truth. A stale, coarse, or manipulated layer yields proofs that verify correctly and mean nothing. Committing to the layer makes that dependency auditable rather than removing it.
- Re-identification is the practical limit rather than the cryptography. Where a region holds few producers of a commodity, the predicate result plus shipment metadata narrows the set quickly.
- Nothing here proves the producer actually harvested from the plot they committed to. That binding is an attestation problem, which is why this pattern composes with attestation rather than replacing it.
- Out of scope: proving volumes, proving chain of custody, and proving the plot is the true origin of the goods.

## Trade-offs

- Polygon containment circuits are costly, and proving time scales with vertex count and layer resolution.
- Raster layers need discretisation, which creates ambiguous cases along cell boundaries that the circuit must resolve conservatively.
- Every layer update creates a re-proving obligation across every affected plot.
- Offline verification requires the verifier to hold the layer commitment locally, which suits customs checkpoints but needs distribution.
- Smallholders rarely run proving infrastructure, so the prover is usually a cooperative or buyer, which reintroduces a trusted intermediary at the point the pattern was meant to remove one.

## Example

- A coffee cooperative holds polygons for 400 member farms and treats the membership list as commercially sensitive.
- The cooperative commits to each polygon and anchors the commitments once.
- For a shipment, it proves that every contributing plot lies outside the flagged region of the published forest layer at the stated baseline.
- The importer and the customs authority verify the proof and learn that all plots passed against layer version `v3`. Neither learns a coordinate, a farm count per shipment, or which members supplied this lot.
- During a later investigation, the authority opens the coordinates for two named plots through the disclosure path, without touching the other 398.

## See also

- [RFC 7946: The GeoJSON Format](https://datatracker.ietf.org/doc/html/rfc7946)
- [EU Forest Observatory global forest cover maps (JRC)](https://forest-observatory.ec.europa.eu/)
- [Jurisdiction: EU / EUDR](../jurisdictions/eu-EUDR.md)
