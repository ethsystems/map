---
title: Private Corporate Bonds
status: ready
primary_domain: Funds & Assets
secondary_domain: Trading
---

## 1) Use Case

Corporate debt issuance where companies raising capital need to hide financing activity from competitors, employees, and market observers. Unlike public/government bonds, corporate bond issuance reveals strategic information about funding needs and potential credit stress.

See [private-bonds.md](private-bonds.md) for general bond patterns. Corporate bonds have specific privacy requirements around hiding funding signals from competitors.

## 2) Additional Context

## 3) Actors

Corporate Issuer · Underwriters · Investors · Crypto-Registry · Regulator

## 4) Problems

### Problem 1: Funding Signal Leakage

Public bond issuance reveals that a company is raising capital, which competitors can use to infer financial stress, acquisition plans, or strategic moves. Employees may also react negatively to perceived funding needs.

**Requirements:**

- **Must hide:** Issuance timing, amounts, pricing negotiations, investor identities (during process)
- **Public OK:** Legal entity identity (post-close where required); existence of debt facility (where mandated by securities law)
- **Regulator access:** Selective disclosure to securities regulators; audit trail for compliance

**Constraints:**

- Securities law disclosure requirements vary by jurisdiction (private placement vs public offering)
- Secondary trading privacy distinct from issuance privacy
- Timeline sensitivity: stealth rounds require extended confidentiality

### Problem 2: Competitive Intelligence from Secondary Trading

Post-issuance trading activity reveals investor sentiment, credit quality perceptions, and potential refinancing needs.

**Requirements:**

- **Must hide:** Trading volumes, prices, counterparties
- **Public OK:** Existence of secondary market activity
- **Regulator access:** Trade reporting for market surveillance

**Constraints:**

- Liquidity requirements for bondholders
- Mark-to-market valuation needs

## 5) Recommended Approaches

See [approach-private-bonds.md](../approaches/approach-private-bonds.md) for general bond architecture. Corporate-specific considerations:

- Extended confidentiality periods for stealth rounds
- Employee communication timing coordination
- Competitor monitoring countermeasures

## 6) Open Questions

- How do disclosure requirements differ between private placements vs public offerings across jurisdictions?
- What's the minimum viable privacy: amounts only, or full term sheet confidentiality?
- How long must confidentiality be maintained post-close?

## 7) Notes And Links

- Related: [private-bonds.md](private-bonds.md) (general bond pattern)
- Related: [private-government-debt.md](private-government-debt.md) (different privacy model - less critical)
- Standards: [ERC-6123](https://ercs.ethereum.org/ERCS/erc-6123) (Draft; derivative/bond lifecycle), [ERC-7573](https://ercs.ethereum.org/ERCS/erc-7573) (Draft; conditional cross-chain DvP, atomicity rests on the decryption oracle, hides no amounts or counterparties; see [pattern](../patterns/pattern-dvp-erc7573.md))
