---
title: Private Stocks
status: ready
primary_domain: Trading
secondary_domain: Funds & Assets
---

## 1) Use Case

Tokenized equity trading where position sizes, trading activity, and ownership stakes must remain confidential. Traditional exchanges face competitive pressure from crypto-native platforms and seek tokenization to create new products while protecting institutional trading strategies from exposure.

## 2) Additional Context

## 3) Actors

Exchanges · Broker-Dealers · Institutional Investors · Retail Investors · Regulators (SEC, FINRA) · Transfer Agents · Custodians

## 4) Problems

### Problem 1: Position and Trading Privacy

Large institutional positions and trading activity on public ledgers reveal investment strategies and enable front-running.

**Requirements:**

- **Must hide:** Position sizes, trading activity, order flow, accumulation/distribution patterns
- **Public OK:** Security existence, total outstanding shares, corporate actions
- **Regulator access:** Beneficial ownership reporting (13F, 13D/G), insider trading surveillance, market manipulation monitoring

**Constraints:**

- Securities law disclosure thresholds (5%, 10% beneficial ownership)
- Short-selling disclosure requirements
- Real-time trade reporting obligations

### Problem 2: Competitive Positioning for Exchanges

Traditional exchanges tokenizing equity need privacy to differentiate from transparent crypto venues and protect institutional order flow.

**Requirements:**

- **Must hide:** Exchange-specific order flow data, market maker positions
- **Public OK:** Aggregate volume, price discovery (NBBO)
- **Regulator access:** Full audit trail for market surveillance

**Constraints:**

- Best execution obligations
- Market data revenue models
- Interoperability with traditional settlement (T+1)

## 5) Recommended Approaches

See [approach-private-trade-settlement.md](../approaches/approach-private-trade-settlement.md) for general settlement architecture. Stock-specific considerations:
- Integration with existing transfer agent infrastructure
- Privacy-preserving beneficial ownership registries
- Hybrid on-chain/off-chain settlement models

## 6) Open Questions

- How do privacy requirements interact with securities law disclosure obligations?
- What's the path from T+1 settlement to instant atomic settlement?
- How to handle corporate actions (dividends, splits) with position privacy?

## 7) Notes And Links

- Related: [private-rwa-tokenization.md](private-rwa-tokenization.md) (general RWA privacy)
- Related: [private-bonds.md](private-bonds.md) (similar DvP requirements)
- Market context: US CLARITY Act (passed the House July 2025) addresses tokenized securities, incl. on-chain equity settlement venues
- Regulatory: SEC, FINRA oversight; beneficial ownership reporting requirements
