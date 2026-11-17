---
title: Private Oracles
status: ready
primary_domain: Data & Oracles
secondary_domain: Trading
---

## 1) Use Case

Price feeds and data sources that protect query privacy - hiding which institutions are requesting specific information. Query patterns reveal trading intent, enabling front-running and information arbitrage. Going forward, the assumption is that oracles should be default private; the entire stack must be end-to-end with no information leakage.

## 2) Additional Context

## 3) Actors

Data Providers · Banks · Trading Firms · Oracle Operators · Regulators · Data regulators · Consent management platforms

## 4) Problems

### Problem 1: Query Privacy and Trading Intent Leakage

Institutions querying price feeds reveal their trading interest. Knowing who is looking at what prices enables front-running, information arbitrage, and competitive intelligence gathering.

**Requirements:**

- **Must hide:** Query source (which institution is asking), query targets (which prices/data), query timing and frequency patterns
- **Public OK:** Aggregated price data, oracle availability, data freshness
- **Regulator access:** Audit trail of queries for market surveillance; selective disclosure mechanisms

**Constraints:**

- Data integrity and availability must not be compromised
- Latency requirements for trading applications
- End-to-end privacy: entire stack must prevent information leakage

### Problem 2: Data Provider Revenue Models

Some institutions (particularly banks) are data providers/originators. Privacy requirements must accommodate both data consumption and data provision roles.

**Requirements:**

- **Must hide:** Who is consuming specific data feeds
- **Public OK:** Data availability, quality metrics
- **Regulator access:** Data provenance, accuracy verification

**Constraints:**

- Data licensing and revenue attribution
- Multiple data provider aggregation
- Conflict of interest management

### Problem 3: Data Provenance & Consent in Oracle Feeds

When oracles aggregate data from multiple institutional sources, data providers need assurance their contributions are used only as agreed. Consumers need provenance guarantees: knowing that the data originates from authorized sources, has not been tampered with, and was aggregated according to a verifiable methodology. Consent receipts (tamper-evident records of data-use agreements) can be anchored on-chain.

**Requirements:**

- **Must hide:** Individual data provider contributions, specific data-use agreement terms
- **Public OK:** Aggregated feed outputs, methodology version, attestation of compliance
- **Regulator access:** Data provenance trail, consent verification, methodology audit

**Constraints:**

- Data protection frameworks (GDPR, sector-specific regulations) apply to oracle data pipelines
- Consent must be granular, revocable, and auditable
- Multiple jurisdictions may apply different rules to the same data feed

## 5) Recommended Approaches

Covered by [Private Oracles](../approaches/approach-private-oracles.md), which compares private information retrieval, TEE-served encrypted queries and batched delivery with anchored provenance, with a 10-row comparison matrix, persona perspectives and a conditional recommendation. Going forward, assumption is oracles should be default private.

Consider:
- Private information retrieval (PIR) techniques
- Encrypted query mechanisms
- Aggregated data delivery (hide individual queries in batches)
- Consent receipts anchored on-chain for data-use agreements

## 6) Open Questions

- Which markets have the highest priority for private oracles (bonds, derivatives, FX, funds)?
- How does private oracle infrastructure integrate with existing data providers?
- What's the performance overhead for query privacy?
- How do data protection frameworks (GDPR) interact with on-chain oracle data provenance?

## 7) Notes And Links

- Cross-cutting concern: affects all use cases requiring external data
- Related: [private-read.md](private-read.md) (query privacy for blockchain state)
- Related: [private-derivatives.md](private-derivatives.md) (oracle-dependent pricing)
- See also: [EPIC map](https://epic-webapp.vercel.app/) (GovTech & EPIC team): consent, data sharing, audit trails
