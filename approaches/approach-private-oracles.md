---
title: "Approach: Private Oracles"
status: draft
last_reviewed: 2026-11-15

use_case: private-oracles
related_use_cases: [private-read, private-derivatives]

primary_patterns: [pattern-private-information-retrieval, pattern-tee-based-privacy, pattern-verifiable-attestation]
supporting_patterns: [pattern-network-anonymity, pattern-mixnet-anonymity, pattern-regulatory-disclosure-keys-proofs]
---

# Approach: Private Oracles

## Problem framing

### Scenario

An institutional trading desk queries price feeds many times a day; the oracle operator sees which institution requested which prices and when, and query patterns reveal trading intent ahead of execution. A bank that is also a data provider needs assurance its feed contributions are used only as licensed, while consuming institutions need proof the data is fresh and from authorized sources without exposing their own request patterns. Regulators need a selective audit path into both sides: query provenance for market surveillance, data provenance for accuracy. The stack is expected to be private end to end, with no layer leaking what the cryptographic layer hides.

### Requirements

- Hide query source, targets, timing and frequency from the oracle operator
- Preserve data integrity, freshness and availability while hiding requests
- Keep provider contributions and license terms confidential between parties
- Anchor data provenance and consent so consumption is provably authorized
- Give regulators a selective path into query and data provenance
- Keep serving latency compatible with trading and settlement use

### Constraints

- Trading-grade latency rules out heavyweight proving on the hot path
- Multiple providers must aggregate without a single trusted party
- Data protection frameworks apply to provider-side pipelines
- Consent must be granular, revocable and auditable
- Oracle infrastructure is shared across many consumers

## Approaches

### Private information retrieval

```yaml
maturity: concept
context: both
crops: { cr: low, o: yes, p: full, s: high }
uses_patterns: [pattern-private-information-retrieval]
```

**Summary:** The consumer fetches a record from the oracle's dataset without the serving infrastructure learning which record was fetched.

**How it works:** The oracle publishes an indexed dataset (price history, reference data). The consumer runs a PIR query: the server processes an encrypted query vector and returns a response the client decrypts to exactly the requested record, while the server learns nothing about which index was read. Batching shares the per-query client work but not the per-query server work: in single-server PIR the server must touch every bit of the database to answer even one query (simplePIR, Henzinger et al., USENIX Security 2023). Consumers share a preprocessing hint instead, roughly 121 MB per 1 GB of dataset, and every dataset update forces the server to repeat part of the preprocessing; for price data, where updates are frequent, that update cost is the binding cost. Pair PIR with a signed snapshot or state-root anchor so freshness and correctness are proven against a separate integrity anchor.

**Trust assumptions:**
- The PIR implementation and its parameters are public and audited
- The dataset is what the operator claims; integrity needs a separate mechanism
- Single-server PIR rests on computational assumptions; multi-server variants need non-colluding operators

**Threat model:**
- Query timing and rate leak trading intent; non-oblivious implementations may also leak the index
- A stale dataset served to a targeted consumer, since integrity is out of PIR's scope
- Collusion between server operators breaks multi-server privacy

**Works best when:**
- The dataset is indexable and the sensitive part is who reads what
- Read patterns are heavy and repeated, amortizing server cost

**Avoid when:**
- The consumer needs proof the served value is fresh and correct, not just unseen
- Real-time tick data under strict latency at very high rates

CROPS rationale: CR low, because the operator holds the only copy of the dataset and can deny any read; this is the single-operator case where the pattern scores CR low. O yes: the implementations are open. P full: the query content is hidden by construction. S high: the assumptions are strong, while side channels and dataset integrity remain open.

### TEE-served encrypted queries

```yaml
maturity: documented
context: both
crops: { cr: low, o: partial, p: partial, s: medium }
uses_patterns: [pattern-tee-based-privacy, pattern-network-anonymity]
```

**Summary:** Queries arrive encrypted to a key held inside a Trusted Execution Environment; the enclave resolves and serves them without the operator reading query content. The operator still sees which consumer connects, and when.

**How it works:** The oracle runs its serving logic inside a TEE. Consumers encrypt queries to the enclave's attested key; the enclave decrypts, looks up, and returns the data, logging only aggregate counters. Remote attestation lets consumers verify the exact code handling their queries before sending them. The operator provisions hardware and network but cannot read query content.

**Trust assumptions:**
- TEE hardware and its attestation infrastructure behave as specified
- The enclave code is open and matches the measured attestation
- Side-channel resistance of the specific hardware generation

**Threat model:**
- Side-channel attacks on the enclave recover query content
- The operator withholds or delays service selectively; availability remains theirs
- Attestation forgery or rollback to older enclave code

**Works best when:**
- Latency matters: lookup is fast, with no proving on the hot path
- The operator is a distinct party whose visibility is the threat

**Avoid when:**
- Threat actors include the hardware supply chain
- Policy requires cryptographic rather than hardware-rooted privacy

CROPS rationale: CR low, because the operator can withhold or delay service selectively, which is exclusion through institutional control under the rubric. O partial: enclave code can be open, while the TEE stack is vendor-controlled. P partial: query content is hidden against the operator, conditional on hardware, but the operator sees which consumer connects, when and how much. S medium: hardware trust and side channels are the weak links.

### Batched delivery with anchored provenance

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: yes, p: partial, s: medium }
uses_patterns: [pattern-verifiable-attestation, pattern-mixnet-anonymity]
```

**Summary:** Feeds publish in fixed rounds with provider provenance attested on-chain, so a consumer's individual queries hide inside a round, while round cadence, volume and the set of contributing providers stay visible.

**How it works:** The oracle publishes feeds in fixed rounds and batches; consumers receive the round bundle or a subset through a mix, so observing the network shows only that an institution consumed a round, not which series it priced off. Each round carries attestations of the contributing providers, the methodology version and freshness; consent receipts anchor provider-consumer data-use agreements on-chain. Auditors and regulators get scoped disclosure into the provenance trail.

**Trust assumptions:**
- Batch composition is large enough that per-institution inference fails
- Attestation signers are honest about provenance and methodology
- Consent receipts match the agreements they record

**Threat model:**
- Small batches let an observer correlate timing with consumption
- A provider's contribution is attributed from aggregated outputs
- Consent receipt drift from the negotiated terms

**Works best when:**
- Many institutions consume the same reference data on similar cadences
- Provider licensing and consent management is the live problem

**Avoid when:**
- A consumer needs bespoke, off-round data at request time

CROPS rationale: CR medium, because the operator runs the rounds and can exclude a consumer, though within a round individual queries are invisible. O yes: the scheme is implementable with open components. P partial: rounds are hidden but cadence and volume leak. S medium: attestation and consent integrity depend on signers and anchor infrastructure.

## Comparison

| Axis | Private information retrieval | TEE-served encrypted queries | Batched delivery with anchored provenance |
|---|---|---|---|
| **Maturity** | concept | documented | documented |
| **Context** | both | both | i2i |
| **CROPS** | CR low · O yes · P full · S high | CR low · O partial · P partial · S medium | CR medium · O yes · P partial · S medium |
| **Trust model** | computational (PIR) or non-colluding servers | hardware attestation and enclave code | signers and batch size |
| **Privacy scope** | query content hidden from server | query hidden from operator; availability theirs | consumption hidden within rounds; provenance selective |
| **Performance** | server touches every bit per query; client downloads a hint (about 121 MB per 1 GB); update cost binds | fast lookups, no hot-path proving | round cadence latency |
| **Operator req.** | Yes (dataset hosting) | Yes (TEE infrastructure) | Yes (round operation) |
| **Cost class** | medium | medium | low |
| **Regulatory fit** | strong on query privacy; needs an integrity layer | strong; audit path is operator-side | strong: provenance and consent are first-class |
| **Failure modes** | side channels; stale data | side channels; selective denial | small-batch correlation; consent drift |

## Persona perspectives

### Business perspective

A trading firm's first concern is that nobody between it and the data learns its interest. Private information retrieval and TEE serving both answer that, and the choice is operational: PIR shifts cost into infrastructure and mathematics, while TEE serving shifts it into hardware the operator already runs. A bank playing both provider and consumer roles needs the batched model's provenance and consent layer regardless, because its licensing terms are the asset, and anchoring consent receipts turns a legal arrangement into an auditable one. Regulators care most about the selective paths, which only the batched model provides natively. A practical desk pairs TEE serving or PIR on the hot path with the batched model's attestation layer for provenance.

### Technical perspective

Private information retrieval moves complexity into infrastructure: dataset packaging, update streams, server cost per query, and parameter choices that must be audited. TEE serving is the shortest path to production but pins the deployment to a hardware vendor and its side-channel record, and availability remains the operator's lever. Batched delivery is simple to build and cheap to run, but its privacy depends on batch size discipline, which is a product decision as much as an engineering one. All three compose: rounds and attestations are orthogonal to how a single query is hidden. Teams should first decide what the operator may be allowed to know, because that single choice picks the sub-approach.

### Legal & risk perspective

Query patterns are commercially sensitive but also exactly what market-surveillance regimes may demand access to, so the design should expose a regulator path scoped to investigations rather than a standing feed. On the provider side, data protection frameworks reach the oracle pipeline when personal or licensed data flows through it, and consent receipts anchored on-chain help only if their terms are precise and revocation is honored in-band. Attestation-based provenance raises an evidentiary question similar to other on-chain records: whether an auditor accepts a cryptographic attestation of methodology in place of source documents. This is not legal advice.

## Recommendation

### Default

For an institutional consumer buying reference data with latency requirements, TEE-served encrypted queries are the default: they hide query content from the operator without proving on the hot path, at the cost of a hardware-vendor trust anchor.

### Decision factors

- If the operator must not learn which record is read, even under a hardware compromise, use private information retrieval instead.
- If licensing and consent management is the binding constraint, add the batched delivery layer with anchored provenance.
- If regulators require query-level audit, keep it behind scoped disclosure rather than standing access.

### Hybrid

Serve the hot path through a TEE or PIR, publish reference data in attested rounds with consent receipts for the provider side, and expose regulators through scoped disclosure keys over the same trail.

## Open questions

1. What PIR server economics are acceptable for trading-grade feeds, and do batched multi-server deployments hold up in practice?
2. How do market-surveillance regimes treat an oracle's query audit trail, and what disclosure granularity will they accept?
3. Can consent receipts be standardized enough that providers and consumers anchor the same agreement identically?
4. How should freshness attestations compose with PIR so integrity is proven, not just assumed?
5. How can contributions from multiple providers aggregate without a single trusted party, while provider attribution and revenue sharing stay private?
