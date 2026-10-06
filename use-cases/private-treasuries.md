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

- **Group treasuries centralize cash through cash pools and in-house banks.** A physical pool sweeps member balances into a pool leader's account; a notional pool leaves balances in place and the bank aggregates them for interest, with no transfer of funds ([OECD Transfer Pricing Guidance on Financial Transactions](https://www.oecd.org/content/dam/oecd/en/publications/reports/2020/02/transfer-pricing-guidance-on-financial-transactions-inclusive-framework-on-beps-actions-4-8-10_278e0cb3/794bcddd-en.pdf), paras 10.111-10.114).
- **Tax authorities already see the financing structure.** Intra-group loans and cash pooling are priced under the arm's-length principle (OECD Transfer Pricing Guidelines, Chapter X, added 2020). The BEPS Action 13 master file describes how the group is financed and names the entities that provide a central financing function; country-by-country reporting applies to groups with consolidated revenue of EUR 750M or more ([Action 13 final report](https://www.oecd.org/content/dam/oecd/en/publications/reports/2015/10/transfer-pricing-documentation-and-country-by-country-reporting-action-13-2015-final-report_g1g58cf0/9789264241480-en.pdf)).
- **Intragroup cash already moves on bank-operated ledgers.** Mitsubishi Corporation uses J.P. Morgan's permissioned Kinexys Digital Payments for intragroup USD cash management across Singapore, London and New York ([March 2026](https://www.jpmorgan.com/payments/newsroom/mitsubishi-cash-management-kinexys)); the same release reports Kinexys average daily volume above USD 5B across all clients (vendor-reported). J.P. Morgan's JPMD deposit token runs on Base, a public Ethereum L2 ([November 2025](https://www.jpmorgan.com/payments/newsroom/jpm-coin-usd-deposit-token-institutional-clients)), where unshielded transfers are visible to any observer.

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
- **Regulator access:** Auditor and tax-authority access to entity-level balances; regulatory capital and liquidity reporting apply only where a group entity is itself a regulated bank or insurer

**Constraints:**

- SEC periodic reporting for US public companies (10-K/10-Q, including the liquidity and capital resources discussion in MD&A under Regulation S-K Item 303)
- Investment policy constraints
- Counterparty credit limits

## 5) Recommended Approaches

See [approach-private-payments.md](../approaches/approach-private-payments.md) for general payment architecture. Treasury-specific considerations:
- Privacy-preserving cash pooling mechanisms
- Integration with [private-money-market-funds.md](private-money-market-funds.md) for yield (see [Approach: Private Money Market Funds](../approaches/approach-private-money-market-funds.md))
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
