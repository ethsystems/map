---
title: Private Repo
status: ready
primary_domain: Trading
secondary_domain: Payments
---

## 1) Use Case

Tokenized repurchase agreements where counterparties and transaction amounts must remain confidential. Repo is short-term secured lending - conceptually similar to DeFi over-collateralized lending, though TradFi has used this instrument for decades. In the U.S. alone, repos outstanding averaged about USD 12.7 trillion a day from July 2025 to March 2026, much of it centrally cleared or settled tri-party ([OFR](https://www.financialresearch.gov/briefs/2026/08/18/closer-look-us-nccbr-market/), 2026-08-18). Privacy prevents counterparty exposure and funding cost arbitrage.

## 2) Additional Context

## 3) Actors

Banks · Clearing Houses · Prime Brokers · Money Market Funds · Central Banks · Regulators

## 4) Problems

### Problem 1: Counterparty Exposure

Repo counterparty relationships reveal funding dependencies and credit relationships. Public exposure enables competitors to target funding sources or exploit perceived credit weakness.

**Requirements:**

- **Must hide:** Counterparty identities, transaction amounts, collateral details, rates
- **Public OK:** Aggregate market statistics, general market rates
- **Regulator access:** Systemic risk monitoring, counterparty exposure aggregation, collateral sufficiency

**Constraints:**

- Same-day (often intraday) settlement requirements
- Collateral eligibility and haircut calculations
- Netting and novation for clearing
- Central bank facility access requirements

### Problem 2: Funding Cost Arbitrage Prevention

Visible funding costs and patterns enable competitors to undercut pricing or front-run funding needs.

**Requirements:**

- **Must hide:** Specific rates obtained, funding frequency, rollover patterns
- **Public OK:** Reference rates (SOFR, repo indices)
- **Regulator access:** Rate surveillance, market manipulation monitoring

**Constraints:**

- Real-time pricing requirements
- Integration with existing repo infrastructure
- Tri-party vs bilateral repo differences

## 5) Recommended Approaches

See [approach-private-trade-settlement.md](../approaches/approach-private-trade-settlement.md) for general settlement architecture. Repo-specific considerations:
- Privacy-preserving collateral verification
- Atomic settlement with privacy (similar to DvP patterns in [approach-private-bonds.md](../approaches/approach-private-bonds.md)); see [Approach: Atomic DvP Settlement](../approaches/approach-dvp-atomic-settlement.md) for the settlement leg
- Integration with existing clearing infrastructure

## 6) Open Questions

- How do intraday settlement requirements interact with privacy mechanisms?
- What's the migration path from centralized repo infrastructure?
- How to handle collateral substitution with position privacy? (Locking a hidden position in favour of a lender is open question 13 in [Approach: Private Money Market Funds](../approaches/approach-private-money-market-funds.md#open-questions))

## 7) Notes And Links

- Related: [private-bonds.md](private-bonds.md) (collateral type, DvP patterns)
- Related: [private-stablecoins.md](private-stablecoins.md) (cash leg privacy)
- Related: [private-money-market-funds.md](private-money-market-funds.md) (MMF shares as confidential collateral; see open question 13 in [Approach: Private Money Market Funds](../approaches/approach-private-money-market-funds.md#open-questions))
- **Domain:** [Trading](../domains/trading.md)
- Market context: U.S. repos outstanding averaged about USD 12.7 trillion a day from July 2025 to March 2026: USD 3.6 trillion centrally cleared, USD 3.1 trillion tri-party, USD 1.1 trillion cleared and tri-party, and USD 5.0 trillion non-centrally cleared bilateral ([OFR Brief 26-05](https://www.financialresearch.gov/briefs/2026/08/18/closer-look-us-nccbr-market/), 2026-08-18)
- Analogy: Conceptually similar to DeFi over-collateralized lending (not flash loans, which are uncollateralized and same-transaction)
