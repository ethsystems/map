---
title: Private Procurement
status: ready
primary_domain: Payments
secondary_domain: Identity & Compliance
---

## 1) Use Case

Procurement and invoicing processes where bid prices, supplier relationships, payment terms, and contract details must remain confidential between parties. Both public and enterprise procurement involve competitive dynamics where premature disclosure distorts markets, enables collusion, or reveals strategic priorities.

## 2) Additional Context

## 3) Actors

Procuring entities (governments, enterprises) · Bidders/suppliers · Auditors · Regulators · Payment processors

## 4) Problems

### Problem 1: Bid Confidentiality and Anti-Collusion

Procurement bids must remain sealed until evaluation to prevent collusion among bidders and manipulation by procuring entities. On-chain procurement creates a transparency problem: if bids are visible, the competitive process breaks down.

**Requirements:**

- **Must hide:** Individual bid amounts, bidder identities (during sealed phase), evaluation criteria weighting
- **Public OK:** Tender existence, general terms, award outcome (post-evaluation)
- **Regulator/auditor access:** Full bid history for procurement compliance review; anti-collusion analysis

**Constraints:**

- Commit-and-reveal timing must be tamper-proof (no late bids, no bid modification after commit)
- Multi-round procurement (negotiated procedures) requires iterative privacy
- Legal frameworks differ by jurisdiction (EU public procurement directives, national rules)

### Problem 2: Invoice and Payment Privacy Between Parties

Invoice amounts, payment terms, and settlement timing reveal pricing leverage, cash flow positions, and supplier dependencies. Competitors observing on-chain payment flows can reverse-engineer cost structures.

**Requirements:**

- **Must hide:** Invoice amounts, payment terms, supplier-specific pricing, payment timing patterns
- **Public OK:** Contract existence (for public procurement), aggregate spending categories
- **Regulator access:** Tax compliance verification, transfer pricing audit

**Constraints:**

- Integration with existing invoicing and ERP systems
- Cross-border VAT and withholding tax reporting
- Dispute resolution requires selective disclosure of contested terms

### Problem 3: Audit Trail Without Exposing Commercial Terms

Regulators and auditors need assurance that procurement followed proper procedures (competitive bidding, conflict-of-interest checks, value-for-money) without accessing every commercial detail. The audit trail must prove procedural compliance while protecting commercially sensitive terms.

**Requirements:**

- **Must hide:** Specific pricing, negotiation history, losing bid details (beyond what law requires)
- **Public OK:** Procedural compliance attestation, aggregate statistics
- **Auditor access:** Selective disclosure of specific procurement steps on demand; proof that evaluation followed declared criteria

**Constraints:**

- Public procurement transparency laws require varying levels of disclosure by jurisdiction
- Audit trails must be tamper-evident and independently verifiable
- Retention periods (typically 5-10 years) require durable privacy guarantees

## 5) Recommended Approaches

Covered by [Private Supply Chain and Procurement](../approaches/approach-private-supply-chain.md), which compares an attested handoff chain, committed logistics data with compliance proofs and sealed procurement on shared rails, with CROPS scores, a comparison matrix, persona perspectives and a conditional recommendation. Key architectural considerations:

- Commit-and-reveal for sealed bids: bidders commit to bids on-chain, reveal after deadline
- Selective disclosure for audit: prove procedural compliance without exposing all commercial terms
- Milestone-based payment release: smart contracts release payment on attested milestones with privacy

## 6) Open Questions

- How do public procurement transparency requirements interact with on-chain bid privacy?
- What commitment schemes best balance bid integrity with operational overhead (two-phase interactions, on-chain storage) at scale?
- How to handle multi-round negotiated procurement with iterative privacy requirements?
- What selective disclosure models satisfy procurement auditors across jurisdictions?

## 7) Notes And Links

- Related patterns: [Commit and Prove](../patterns/pattern-commit-and-prove.md), [L2 Encrypted Offchain Audit](../patterns/pattern-l2-encrypted-offchain-audit.md), [Verifiable Attestation](../patterns/pattern-verifiable-attestation.md)
- Related use cases: [private-payments.md](private-payments.md) (payment privacy), [private-supply-chain.md](private-supply-chain.md) (shares the supply chain and procurement approach), [private-government-debt.md](private-government-debt.md) (sealed auction parallels)
- See also: [EPIC map](https://epic-webapp.vercel.app/) (GovTech & EPIC team): procurement & invoicing, audit trails, grants tracking
