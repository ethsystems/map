---
title: Private Read
status: ready
primary_domain: Data & Oracles
secondary_domain: Trading
---

## 1) Use Case

Blockchain query privacy for institutions interacting with public blockchains. Institutions monitoring addresses, checking balances, or querying transaction history leak competitive intelligence through their read patterns. Query metadata reveals trading intent, portfolio composition, and counterparty relationships.

Private RPCs are one implementation approach; the broader scope includes query pattern obfuscation, timing analysis prevention, and read-path privacy across the entire blockchain interaction stack.

## 2) Additional Context

## 3) Actors

Institutions (banks, trading firms, asset managers) · RPC Providers · Node Operators · MEV Searchers · Regulators

## 4) Problems

### Problem 1: Competitive Intelligence Leakage from Blockchain Reads

Institutions querying blockchain state reveal their interests and positions. RPC providers, node operators, and network observers can infer trading strategies from query patterns.

**Requirements:**

- **Must hide:** Query source (which institution is asking), query targets (which addresses/contracts), query timing and frequency patterns
- **Public OK:** Aggregated network statistics; general query volume
- **Regulator access:** Audit trail of institutional queries for compliance; selective disclosure mechanisms

**Constraints:**

- Latency requirements for trading applications
- Many institutions do not run own infrastructure
- MEV protection increasingly critical

### Problem 2: MEV Extraction from Read Patterns

Query patterns preceding transactions enable MEV extraction. Knowing what an institution is looking at reveals imminent trades.

**Requirements:**

- **Must hide:** Pre-transaction query activity; address monitoring patterns
- **Public OK:** General blockchain state (post-query)
- **Regulator access:** Query logs for market surveillance

**Constraints:**

- Real-time trading requirements
- Integration with existing trading infrastructure

## 5) Recommended Approaches

See [Approach: Private Read](../approaches/approach-private-read.md): institution-run nodes where the institution can operate them, a TEE-hosted private RPC otherwise, and obfuscated aggregate reads as a research path.

## 6) Open Questions

- What's the threat model: RPC provider, network observer, or both?
- How do private reads integrate with private transactions end-to-end?
- Cost/latency trade-offs for different privacy levels?

## 7) Notes And Links

- Cross-cutting concern: affects all use cases where institutions read blockchain state
- Related: [private-oracles.md](private-oracles.md) (data feed query privacy)
- RFP: [RFP-private-reads](../rfps/rfp-private-reads.md)
