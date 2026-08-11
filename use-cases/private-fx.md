---
title: Private FX
status: ready
primary_domain: Payments
secondary_domain: Trading
---

## 1) Use Case

Cross-border foreign exchange settlement on blockchain where trading intent, positions, and execution details must remain confidential. FX is one of the largest financial markets, and the vast majority of stablecoins are USD-pegged, making cross-currency settlement a natural blockchain application. Institutions require privacy to prevent front-running and protect trading strategies.

## 2) Additional Context

## 3) Actors

Banks · Payment Networks · Corporate Treasury · Liquidity Providers · Regulators · Stablecoin Issuers

## 4) Problems

### Problem 1: Trading Intent and Front-Running Prevention

FX markets are highly competitive with tight spreads. Leaked trading intent enables front-running and adverse price movements.

**Requirements:**

- **Must hide:** Order size, direction, timing, counterparty relationships
- **Public OK:** General market liquidity, executed trade existence (after settlement)
- **Regulator access:** Trade reporting for market surveillance; sanctions screening

**Constraints:**

- Real-time settlement requirements
- Liquidity fragmentation across stablecoin venues
- Regulatory reporting obligations (EMIR, Dodd-Frank)

### Problem 2: Cross-Border Settlement Privacy

International payments reveal business relationships and transaction patterns that competitors can exploit.

**Requirements:**

- **Must hide:** Payment amounts, sender/receiver identities, payment purpose
- **Public OK:** Settlement finality confirmation
- **Regulator access:** AML/CFT monitoring; cross-border transaction reporting

**Constraints:**

- Multi-jurisdictional compliance requirements
- Correspondent banking relationships
- SWIFT alternative positioning

## 5) Recommended Approaches

See [approach-private-trade-settlement.md](../approaches/approach-private-trade-settlement.md) for general settlement architecture. FX-specific considerations:
- Privacy-preserving PvP (Payment vs Payment) settlement
- Integration with [approach-private-bonds.md](../approaches/approach-private-bonds.md) DvP patterns for cash leg
- Stablecoin privacy patterns from [private-stablecoins.md](private-stablecoins.md)

## 6) Open Questions

- How does privacy interact with real-time gross settlement requirements?
- What's the role of central bank digital currencies vs private stablecoins?
- How to handle multi-currency atomic swaps with privacy?

## 7) Notes And Links

- Related: [private-stablecoins.md](private-stablecoins.md) (settlement currency privacy)
- Related: [private-payments.md](private-payments.md) (payment privacy patterns)
- Market context: Payment networks exploring Ethereum-based SWIFT alternatives
- Standards: ERC-7573 (atomic settlement), ISO 20022 (messaging)
