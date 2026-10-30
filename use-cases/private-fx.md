---
title: Private FX
status: ready
primary_domain: Payments
secondary_domain: Trading
---

## 1) Use Case

Cross-border foreign exchange settlement on blockchain where trading intent, positions, and execution details must remain confidential. FX is one of the largest financial markets ([$9.6 trillion per day in April 2025](https://www.bis.org/press/p250930.htm), BIS Triennial Survey), and [99.4% of fiat-backed stablecoins by market value are USD-pegged](https://www.bis.org/publ/arpdf/ar2026e3.htm) (BIS Annual Economic Report, June 2026), making cross-currency settlement a natural blockchain application. Institutions require privacy to prevent front-running and protect trading strategies.

## 2) Additional Context

- The US dollar is on one side of 89% of FX trades ([BIS Triennial Survey, April 2025](https://www.bis.org/press/p250930.htm)).
- Off-chain, [CLS](https://www.cls-group.com/products/settlement/clssettlement/) settles FX payments in 18 currencies through payment-versus-payment (PvP) to remove settlement risk.
- Swift [announced a blockchain-based shared ledger in September 2025](https://www.theblock.co/post/372706/swift-blockchain-ledger-consensys) with a Consensys prototype, and launched an initial live pilot of cross-border tokenized-deposit payments with 17 banks in [July 2026](https://www.theblock.co/post/407687/swift-launches-blockchain-ledger-for-tokenized-deposit-pilot-with-17-banks); final settlement still runs through existing systems.

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
- Regulatory reporting obligations for FX derivatives: forwards, swaps and options are reportable under EMIR ([Art. 9](https://eur-lex.europa.eu/eli/reg/2012/648/oj); currency derivatives per MiFID II Annex I C(4)) and Dodd-Frank (OTC FX options are swaps under the [CFTC/SEC product definitions](https://www.cftc.gov/LawRegulation/FederalRegister/FinalRules/2012-18003.html); the [Treasury FX determination, 2012](https://www.govinfo.gov/content/pkg/FR-2012-11-20/html/2012-28319.htm) exempts FX swaps and forwards from clearing but not trade reporting); spot FX (delivery within two trading days for major pairs, [Delegated Regulation (EU) 2017/565 Art. 10](https://eur-lex.europa.eu/eli/reg_del/2017/565/oj); T+2 in US practice.
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
- Privacy-preserving PvP (Payment vs Payment) settlement: [Private PvP via ERC-7573](../patterns/pattern-private-pvp-stablecoins-erc7573.md) (amount privacy only when the escrow contracts run inside a shielded layer)
- Cross-network settlement options and their recovery paths in [Approach: Atomic DvP Settlement](../approaches/approach-dvp-atomic-settlement.md)
- Stablecoin privacy patterns from [private-stablecoins.md](private-stablecoins.md)

To keep trading intent out of the public mempool before inclusion (Problem 1), see [Approach: Private Transaction Broadcasting](../approaches/approach-private-broadcasting.md).

## 6) Open Questions

- How does privacy interact with real-time gross settlement requirements?
- What's the role of central bank digital currencies vs private stablecoins?
- How to handle multi-currency atomic swaps with privacy?

## 7) Notes And Links

- Related: [private-stablecoins.md](private-stablecoins.md) (settlement currency privacy)
- Related: [private-payments.md](private-payments.md) (payment privacy patterns)
- Market context: Swift's own blockchain-based shared ledger ([tokenized-deposit pilot, July 2026](https://www.theblock.co/post/407687/swift-launches-blockchain-ledger-for-tokenized-deposit-pilot-with-17-banks))
- Standards: [ERC-7573](https://ercs.ethereum.org/ERCS/erc-7573) (Draft; conditional settlement, atomicity rests on the decryption oracle, hides no amounts or counterparties, no FX rate check), [ISO 20022](https://www.iso20022.org/) (messaging)
