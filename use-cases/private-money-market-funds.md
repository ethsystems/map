---
title: Private Money Market Funds
status: ready
primary_domain: Funds & Assets
secondary_domain: Payments
---

## 1) Use Case

Tokenized money market funds providing yield-bearing treasury management for institutions. Unlike stablecoins (which typically don't earn yield), money market funds are productive assets generating returns. Institutions prefer MMFs over non-yield stablecoins for treasury management but require privacy for position sizes and yield strategies.

## 2) Additional Context

- **Holders differ in what they want disclosed.** Reserve holders (for example, stablecoin issuers backing their token) publish positions on purpose; trading firms want the opposite. Confidentiality needs a per-holder choice.
- **The transfer agent keeps the register of record** and must be able to rebuild it in full: eligibility, holder counts, sanctions screening, tax.
- **Most tokenized MMFs on Ethereum today keep a stable USD 1 NAV** and pay yield as new shares; floating-NAV funds pay it through the share price.
- **Collateral use is the likely next step.** Tokenized MMF shares already serve as collateral and as stablecoin reserves in the clear. Using them as collateral confidentially is a future extension (see [Private Repo](private-repo.md) and the approach card's open question 13).

## 3) Actors

Asset Managers · Institutional Investors · Banks · Custodians · Regulators · NAV Calculation Agents

## 4) Problems

### Problem 1: Position and Strategy Privacy

Large MMF positions reveal treasury management strategies and cash reserves. Competitors can infer financial health and business plans from position sizes.

**Requirements:**

- **Must hide:** Position sizes, subscription/redemption timing, yield optimization strategies
- **Public OK:** Fund NAV, total AUM, portfolio composition (at aggregate level)
- **Regulator access:** Investor concentration monitoring, liquidity stress testing data, SEC reporting

**Constraints:**

- Daily NAV calculations
- SEC Rule 2a-7 compliance (for registered US funds); offering rules such as eligibility and investor-count limits for private funds
- Liquidity requirements for redemptions

### Problem 2: Redemption Pattern Privacy

Redemption patterns signal liquidity needs or market views. Large redemptions can trigger runs if publicly visible.

**Requirements:**

- **Must hide:** Individual redemption requests, timing, amounts
- **Public OK:** Aggregate fund flows (delayed)
- **Regulator access:** Liquidity monitoring, stress scenario analysis

**Constraints:**

- Same-day or T+1 redemption requirements
- Fee provisions, and gates where the regime has them (ESMA MMFR; removed from Rule 2a-7 in 2023)
- Systemic risk monitoring obligations

## 5) Recommended Approaches

See detailed solution architecture and trade-offs in [**Approach: Private Money Market Funds**](../approaches/approach-private-money-market-funds.md). Key considerations:

- Privacy-preserving subscription/redemption mechanisms
- Confidential NAV sharing with authorized parties
- Integration with custody and settlement infrastructure

## 6) Open Questions

- How is yield attributed across entry-NAV cohorts in floating-NAV funds without revealing positions?
- Can the confidential set be shared with stablecoins used as the cash leg, and how are per-asset rules enforced then?
- What does each crossing between public and confidential holdings leak, and how large must the confidential set be?

## 7) Notes And Links

- Related: [private-stablecoins.md](private-stablecoins.md) (non-yield alternative; settlement cash)
- Related: [private-treasuries.md](private-treasuries.md) (corporate treasury use of MMFs)
- Differentiation: Unlike stablecoins, money market funds earn yield. Institutions prefer MMFs over non-yield stablecoins for treasury management.
- Market context: Tokenized treasuries/MMFs are among the largest RWA categories outside stablecoins
