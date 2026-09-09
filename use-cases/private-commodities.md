---
title: Private Commodities
status: ready
primary_domain: Trading
secondary_domain: Funds & Assets
---

## 1) Use Case

Tokenized commodity trading where position sizes, trading strategies, and physical delivery arrangements must remain confidential. Gold is the primary tokenized commodity today, typically using synthetic/digital twin models. Trading houses require privacy to protect proprietary strategies and prevent market manipulation through position exposure.

## 2) Additional Context

## 3) Actors

Trading Houses · Commodity Exchanges · Institutional Investors · Physical Custodians · Regulators (CFTC) · Oracles (price feeds, physical verification)

## 4) Problems

### Problem 1: Position Exposure and Market Manipulation

Large commodity positions on public ledgers enable market manipulation and competitive intelligence gathering. Position size disclosure can move markets.

**Requirements:**

- **Must hide:** Position sizes, accumulation/liquidation patterns, physical delivery arrangements
- **Public OK:** Token existence, total supply, custody attestations
- **Regulator access:** Position limits monitoring, market manipulation surveillance, physical delivery tracking

**Constraints:**

- CFTC position reporting requirements
- Physical delivery coordination
- Custody and insurance verification

### Problem 2: Trading Strategy Protection

Commodity trading strategies depend on information asymmetry. Public trading activity eliminates competitive advantage.

**Requirements:**

- **Must hide:** Order flow, execution timing, counterparty relationships
- **Public OK:** Aggregate market volume, settlement prices
- **Regulator access:** Trade reporting, anti-manipulation monitoring

**Constraints:**

- Real-time price discovery requirements
- Margin and collateral management
- Cross-exchange position aggregation

## 5) Recommended Approaches

See [approach-private-trade-settlement.md](../approaches/approach-private-trade-settlement.md) for general settlement architecture. Commodity-specific considerations:
- Privacy-preserving custody attestations (proof of reserves without position disclosure)
- Integration with physical commodity tracking
- Synthetic vs physically-backed privacy trade-offs

## 6) Open Questions

- How do privacy requirements differ for physically-backed vs synthetic commodity tokens?
- What oracle infrastructure needed for private commodity price feeds?
- How to handle position limits compliance with position privacy?

## 7) Notes And Links

- Related: [private-oracles.md](private-oracles.md) (price feed privacy)
- Related: [private-stocks.md](private-stocks.md) (similar trading privacy requirements)
- Market context: Gold is primary tokenized commodity; digital twin models common
- Regulatory: CFTC oversight, position limits, delivery requirements
