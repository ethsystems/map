---
title: "Approach: Private Supply Chain and Procurement"
status: draft
last_reviewed: 2026-11-15

use_case: private-supply-chain
related_use_cases: [private-procurement]

primary_patterns: [pattern-verifiable-attestation, pattern-commit-and-prove, pattern-l2-encrypted-offchain-audit]
supporting_patterns: [pattern-regulatory-disclosure-keys-proofs, pattern-zk-proof-systems]
---

# Approach: Private Supply Chain and Procurement

## Problem framing

### Scenario

A mid-size pharmaceutical manufacturer with three contract manufacturers and two hundred distributors must prove to pharmacies, customs and regulators that every batch is genuine and compliant, without letting competitors read its supplier network, volumes or routing. The same institution runs procurement: sealed-bid tenders where premature disclosure breaks the competitive process, and invoices whose payment patterns reveal cost structures. Today the provenance trail lives in ERP systems each party controls separately, and an audit is a document request. Both processes share one shape: many counterparties produce evidence per handoff or bid, evidence must be verifiable at the point of use, and the full record must be reconstructible for an auditor or regulator while staying invisible to any single commercial party.

### Requirements

- Prove origin, handling and compliance at each handoff without exposing supplier identities
- Verify authenticity at the point of use: pharmacy dispensing, customs checkpoints
- Keep bid amounts and bidder identities sealed until evaluation closes
- Release payment on attested milestones without exposing commercial terms
- Give regulators a selective path into the full record for a specific batch or tender
- Integrate with existing ERP and serialization systems (EU FMD, US DSCSA)

### Constraints

- Attestation standards differ across jurisdictions and industries
- Customs checks need near-real-time latency; distribution points may be offline
- Public procurement disclosure law varies by jurisdiction
- Audit trails must be tamper-evident and survive 5-10 year retention
- Multi-party chains mean no single operator owns the whole process
- Dispute resolution needs selective disclosure of contested terms

## Approaches

### Attested handoff chain

```yaml
maturity: documented
context: both
crops: { cr: medium, o: yes, p: partial, s: medium }
uses_patterns: [pattern-verifiable-attestation, pattern-regulatory-disclosure-keys-proofs, pattern-l2-encrypted-offchain-audit]
```

**Summary:** Each handoff or procurement milestone is attested on-chain with a minimal public payload; commercial detail stays encrypted off-chain, disclosed selectively to auditors and regulators.

**How it works:** Each party attests its step (shipped, received, inspected, cleared; bid received, milestone delivered) as a signed, schema-defined record anchored on a public attestation layer. The on-chain record carries only public-safe fields: product type, attestation type, timestamp, and a commitment to the detail. Commercial detail (supplier identity, price, volume) lives in an encrypted off-chain store referenced by the attestation. Auditors and regulators receive scoped disclosure through disclosure keys and proofs rather than by reading the chain, and the encrypted trail supports later compliance monitoring.

**Trust assumptions:**
- Each attesting party signs with keys under its own operational control
- The attestation schema and its registry are maintained by a consortium or standards body
- Off-chain encrypted stores depend on the storage operator and its key management
- Regulator disclosure relies on the disclosure-key holder acting as agreed
- Settlement assumes a sufficiently decentralized chain

**Threat model:**
- A competitor reads public attestations and runs network analysis; minimal payloads are the mitigation
- A party refuses to attest (holdout); this is process risk, not on-chain forgery
- Key compromise lets an attacker issue attestations in a party's name
- Disclosure keys reused beyond the scoped request

**Works best when:**
- Parties already serialize and track batches in ERP systems
- The public-safe facts are themselves valuable downstream (authenticity, recall status)
- Many jurisdictions are involved, so a common attestation schema beats per-jurisdiction disclosure rules

**Avoid when:**
- Even the fact of a transaction is commercially sensitive
- Verification must run fully offline at the point of use

CROPS rationale: CR medium, because participation is permissioned to supply chain members and a member can be excluded by the consortium, but no single party controls another's attestations and readers verify without permission. O yes: attestation layers and schemas are open source and records are portable. P partial: payloads are minimized, but attestation existence and timing are public and disclosure is structured. S medium: signature security is sound, while key management and consortium governance concentrate operational risk.

### Committed logistics data with compliance proofs

```yaml
maturity: concept
context: both
crops: { cr: high, o: partial, p: full, s: medium }
uses_patterns: [pattern-commit-and-prove, pattern-zk-proof-systems]
```

**Summary:** Batch and invoice detail never leaves the holder in readable form; zero-knowledge proofs assert compliance claims against commitments anchored on-chain.

**How it works:** Each party commits to its logistics data, or to its bid, on-chain as a binding commitment. When a downstream check is needed, the holder generates a zero-knowledge proof that the committed data satisfies the claim (batch within temperature range; unit serialized under DSCSA; bid meets the declared criteria) without opening the data. Verification is a single check on-chain or off-chain, and the verifier learns the claim, not the data.

**Trust assumptions:**
- The proof system and circuits are public and auditable
- Claim semantics (what "compliant" means) are encoded in circuits all parties can review
- Off-chain source data must survive; its loss breaks future proofs
- Proving infrastructure availability is an operational dependency

**Threat model:**
- Circuit bugs prove false claims; third-party circuit audit is the mitigation
- Mis-binding commitments let a party substitute data after the fact
- Prover-side data leakage during proof generation (side channels, hosted provers)
- A regulator demands raw data the system was designed not to reveal

**Works best when:**
- The compliance claim is boolean and stable (genuine, in-range, serialized)
- Anti-counterfeiting intensity is high (pharmaceuticals, luxury goods)
- Competitor inference from raw flows is the dominant harm

**Avoid when:**
- Regulators require the raw record itself by statute, not just claims
- Parties cannot operate proving infrastructure at unit scale (millions of units per year)

CROPS rationale: CR high, because verification is permissionless once claims are defined and exclusion requires blocking the chain rather than an operator's decision. O partial: the proof-system stacks are open, but circuits and claim semantics are authored per deployment. P full: nothing beyond the claim is revealed by design. S medium: cryptographic assumptions are strong; circuit correctness and off-chain data custody are the weak links.

### Sealed procurement on shared rails

```yaml
maturity: concept
context: i2i
crops: { cr: medium, o: yes, p: partial, s: medium }
uses_patterns: [pattern-commit-and-prove, pattern-l2-encrypted-offchain-audit]
```

**Summary:** The procurement leg: bids committed sealed on-chain, opened by the contract at the deadline, evaluation evidenced against declared criteria, and payment released on attested milestones.

**How it works:** Bidders commit to their bids on-chain before a deadline; the contract enforces commit and reveal phase timing so late bids and post-commit modification are impossible. After the deadline, bids open, on-chain or to the evaluation committee under disclosure keys depending on the jurisdiction. Evaluation outcomes are attested against the declared criteria, so the trail shows procedure was followed without revealing losing-bid detail. Payment runs on attested milestones with amounts visible only to the parties and their auditors.

**Trust assumptions:**
- Bidders control commitment timing and entropy
- The evaluating entity operates the opening step honestly
- Threshold-encrypted or key-managed openings where bids must not reveal publicly

**Threat model:**
- Block proposers censor commits inside the bid window
- Bid sniping at the phase boundary; ending the commit phase before the reveal phase mitigates it
- Collusion signaled through timing patterns rather than amounts

**Works best when:**
- The tender is high-stakes enough to justify a two-phase ceremony
- Public procurement law allows a sealed phase, as most regimes do

**Avoid when:**
- Procurement volume is high-frequency and low-value, where ceremony overhead dominates

CROPS rationale: CR medium, because commit windows are enforced by contract but block proposers can censor commits, which needs inclusion-list style mitigations. O yes: the pattern is implementable entirely with open contracts. P partial: commitments hide amounts while bid timing and count stay observable. S medium: tamper-proof timing holds only if the opening step is operated honestly.

## Comparison

| Axis | Attested handoff chain | Committed data with compliance proofs | Sealed procurement on shared rails |
|---|---|---|---|
| **Maturity** | documented | concept | concept |
| **Context** | both | both | i2i |
| **CROPS** | CR medium · O yes · P partial · S medium | CR high · O partial · P full · S medium | CR medium · O yes · P partial · S medium |
| **Trust model** | signed attestations per party, consortium schema | cryptographic proofs, no data disclosure | contract-enforced commit/reveal timing |
| **Privacy scope** | payloads minimized, existence public | claim-only disclosure, private by design | amounts hidden, timing observable |
| **Performance** | not measured | proof generation per check, not measured | two-phase ceremony per tender |
| **Operator req.** | Yes (off-chain store, disclosure keys) | Yes (proving infrastructure, data custody) | No (contract-run phases) |
| **Cost class** | low | high | medium |
| **Regulatory fit** | strong: selective disclosure to regulators | strong where claims suffice; weak where raw record mandated | strong: matches sealed-bid statutes |
| **Failure modes** | key compromise; consortium holdout | circuit bugs; source data loss | commit censorship; dishonest opening |

## Persona perspectives

### Business perspective

For an institution whose competitive edge is the supplier network or bid strategy, the attested handoff chain is the entry point: it builds on attestation infrastructure that already runs in production and delivers downstream value immediately, authenticity checks and recall status, while pricing and volumes stay off-chain. The committed-data approach buys strictly more privacy, since competitors learn nothing beyond claims, and fits high-intensity anti-counterfeiting segments where the proving spend pays for itself. Procurement teams with recurring tender cycles get the most from the sealed-procurement leg, which removes the two classic failures of on-chain procurement: bid leakage and after-the-fact modification. Most institutions will run the first, pilot the second, and adopt the third where tender volume justifies the ceremony.

### Technical perspective

The attested handoff chain is operationally light: signing keys, a schema registry, an encrypted off-chain store; the hard parts are schema governance across jurisdictions and disclosure-key custody. The committed-data approach is a different engineering class: circuit design, proof infrastructure, and per-check latency at unit scale, since a dispensing check must complete in seconds, plus custody discipline for source data whose loss breaks future proofs. Sealed procurement is contract engineering with careful phase semantics; known risks are commit-window censorship and timing side channels, both addressable with existing mempool-privacy patterns at some operational cost. Sequence the work: attestation schema first, proofs second, procurement ceremonies third.

### Legal & risk perspective

Two questions dominate. First, disclosure: serialization mandates (EU FMD, US DSCSA) give regulators access rights a privacy-preserving design must satisfy by construction, so scoped disclosure paths need review against each authority's actual access procedure, and a design that proves claims may not satisfy a statute demanding the raw record. Second, procurement law: sealed-bid rules generally permit commitment schemes, but the evidentiary value of an on-chain trail depends on the attestation's legal weight, which varies by jurisdiction, so test whether audit authorities accept cryptographic attestation in place of document production. Retention obligations, commonly five to ten years, also constrain key management: keys that make data unreadable can conflict with mandated retention. This is not legal advice.

## Recommendation

### Default

For the baseline institutional case, moderate volume and standard compliance, the attested handoff chain is the default: it delivers verifiable provenance and sealed audit trails on infrastructure that runs today, and leaves room to deepen privacy where a specific segment demands it.

### Decision factors

- If competitors inferring sourcing strategy is the dominant harm, add the committed-data approach for the sensitive segments.
- If procurement is a major recurring process, run the sealed-procurement leg alongside the chain.
- If a regulator requires the raw record by statute, the committed-data approach cannot carry that leg alone.

### Hybrid

Anchor attestations on-chain for public-safe facts, run committed-data proofs for the commercial heart of the operation (pricing, volumes, routing), and keep sealed procurement as its own ceremony sharing the same attestation and audit rails.

## Open questions

1. What is the minimum viable attestation standard for cross-jurisdictional supply chain privacy, and which body maintains it?
2. How do serialization mandates (EU FMD, US DSCSA) treat claim-level proofs in place of raw unit-level disclosure?
3. What proving latency is achievable at unit scale for point-of-dispensing verification?
4. Which selective-disclosure models satisfy procurement auditors across the major jurisdictions?
5. How should dispute resolution work when provenance data is privacy-protected and the counterparty is the disputing party?
