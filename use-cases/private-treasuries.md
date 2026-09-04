---
title: Private Treasuries
status: ready
primary_domain: Payments
secondary_domain: Funds & Assets
---

## 1) Use Case

Corporate treasury operations on-chain where multi-entity organizations need to manage internal cash flows without revealing financing activity to competitors. Large corporations move billions daily between subsidiaries, and these internal transfers can reveal business strategy, regional performance, and capital allocation decisions.

Note: This is corporate treasury management, NOT government securities (treasuries). Government debt is covered in [private-government-debt.md](private-government-debt.md).

## 2) Additional Context

## 3) Actors

Corporate Treasury · Subsidiaries · Banks · Auditors · Regulators · Tax Authorities

## 4) Problems

### Problem 1: Internal Transfer Privacy

Inter-entity transfers reveal capital allocation, subsidiary performance, and strategic priorities. Competitors monitoring public chains can infer business health and plans.

**Requirements:**

- **Must hide:** Transfer amounts, entity identities (internal structure), timing patterns, cash pooling arrangements
- **Public OK:** Corporate existence, banking relationships (where publicly disclosed)
- **Regulator access:** Tax authority reporting, transfer pricing documentation, audit trails

**Constraints:**

- Multi-jurisdictional operations
- Transfer pricing regulations
- Intercompany loan documentation
- Real-time cash visibility requirements (internal)

### Problem 2: Cash Position Confidentiality

Aggregate cash positions and treasury investment strategies are competitively sensitive. Public visibility enables predatory competitor behavior.

**Requirements:**

- **Must hide:** Cash balances by entity, investment allocations, yield optimization strategies
- **Public OK:** Audited financial statements (periodic, aggregated)
- **Regulator access:** Regulatory capital calculations, liquidity reporting

**Constraints:**

- SEC reporting requirements (for public companies)
- Investment policy constraints
- Counterparty credit limits

## 5) Recommended Approaches

See [approach-private-payments.md](../approaches/approach-private-payments.md) for general payment architecture. Treasury-specific considerations:
- Privacy-preserving cash pooling mechanisms
- Integration with [private-money-market-funds.md](private-money-market-funds.md) for yield
- Multi-entity identity management with selective disclosure

## 6) Open Questions

- How to maintain internal visibility while hiding from external observers?
- What's the relationship to existing treasury management systems (TMS)?
- How do transfer pricing audits work with transaction privacy?

## 7) Notes And Links

- Related: [private-money-market-funds.md](private-money-market-funds.md) (treasury investment vehicle)
- Related: [private-payments.md](private-payments.md) (external payment privacy)
- Related: [private-stablecoins.md](private-stablecoins.md) (settlement currency)
- Note: NOT government treasuries - see [private-government-debt.md](private-government-debt.md) for sovereign/municipal debt
- Market context: Large corporations move billions daily between entities
