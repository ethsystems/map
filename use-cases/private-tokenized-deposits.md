---
title: Private Tokenized Deposits
status: ready
primary_domain: Payments
---

## 1) Use Case

Banks represent customer deposits as tokens on a shared ledger so that their customers can pay each other, and customers of other banks, with on-ledger settlement. The deposit stays a liability of the issuing bank, booked in its core banking system, and the token is a claim on it. The banks sharing the ledger are often competitors, so each needs its customers' payments hidden from the other banks and from the public, while the core system, the ledger and the supervisor's view stay in agreement.

Unlike [private stablecoins](private-stablecoins.md), the issuer is the depositor's own bank and the token is a deposit rather than a reserve-backed coin. The privacy patterns overlap; holder eligibility, the interbank settlement leg and the link to the core banking system are specific to deposits.

## 2) Additional Context

- **Stablecoin laws leave deposits out.** The US GENIUS Act excludes deposits, including deposits recorded using distributed ledger technology, from its definition of a payment stablecoin ([§ 2(22)(B)(ii)](https://www.govinfo.gov/content/pkg/PLAW-119publ27/html/PLAW-119publ27.htm)). MiCA does not apply to crypto-assets that qualify as deposits ([Art. 2(4)(b)](https://eur-lex.europa.eu/eli/reg/2023/1114/oj)).
- **Banks already issue deposit tokens.** J.P. Morgan's JPMD deposit token runs on Base, a public Ethereum L2 ([November 2025](https://www.jpmorgan.com/payments/newsroom/jpm-coin-usd-deposit-token-institutional-clients)), where unshielded transfers are visible to any observer. Swift launched a live pilot of cross-border tokenized-deposit payments with 17 banks in [July 2026](https://www.theblock.co/post/407687/swift-launches-blockchain-ledger-for-tokenized-deposit-pilot-with-17-banks); final settlement still runs through existing systems.
- **Interbank settlement against tokenized central bank money is in testing.** In BIS [Project Agorá](https://www.bis.org/project/agora), banks issue tokenized deposits on a shared ledger and manage tokenized central bank reserves on each central bank's ledger. A real-value test in July 2026 moved about CHF 800,000 across 28 financial institutions and central banks. Data privacy is among the rule areas the project is still examining.
- **A public demonstration covers the bank-side flow.** Gateway.fm's [tokenized deposit issuance and redemption](https://gateway.fm/case-studies/tokenized-deposit-issuance-and-redemption/) demonstration runs on [Miden](../vendors/miden.md), with [Gateway Open Privacy Suite](../vendors/gateway-open-privacy-suite.md) for access control, against a simulated bank. The core system places a hold on the deposit before the claim is minted, redemption burns the claim and releases the held funds, and a reconciliation run compares ledger supply with the core liability. The ledger stores commitments rather than balances and amounts. The operator co-signs every movement and can list holders; the transaction graph, total supply and redemption amounts stay visible. The demonstration covers a single bank with no interbank transfer, and the page links no code.

## 3) Actors

Issuing bank · Receiving bank · Depositors (corporate, retail) · Ledger operator · Interbank settlement provider (central bank or settlement bank) · Supervisor · Auditor

## 4) Problems

### Problem 1: Payment Privacy on a Shared Ledger

When several banks and their customers share one ledger, each transfer exposes payer, payee, amount and balances to competing banks on the same ledger and to any public observer. The issuing bank already sees its own customers' activity in its core system, so the privacy requirement is towards the other participants and the public.

**Requirements:**

- **Must hide:** amounts, balances, payer and payee identities, and the payment graph from other banks and from the public
- **Public OK:** existence of a transaction; contract code; validity proofs
- **Issuer access:** the issuing bank sees its own customers' transfers, including the payee, as it does for account payments today
- **Regulator access:** supervisory and AML/CFT access through scoped, logged and revocable disclosure; Travel Rule originator and beneficiary information for cross-border transfers ([FATF R.16, revised June 2025](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/update-Recommendation-16-payment-transparency-june-2025.html))

**Constraints:**

- Holders are customers the issuing bank has onboarded; a transfer to an ineligible holder fails
- The issuing bank can freeze or block a holding where law requires, without a standing view of other banks' customers
- Throughput and finality are measured on the chosen stack, not taken from vendor claims

### Problem 2: Transfers Between Banks

A tokenized deposit is a claim on one bank. When a customer of bank A pays a customer of bank B, bank A's liability becomes bank B's, and the two banks settle with each other, usually in central bank money. On a shared ledger, that interbank leg reveals bilateral flows between banks unless it is shielded too.

**Requirements:**

- **Must hide:** interbank settlement amounts, and the customer payments behind them, from banks that are not party to them
- **Public OK:** existence of the settlement arrangement
- **Settlement:** the customer leg and the interbank leg settle atomically, or the design names who carries the risk between them

**Constraints:**

- Central bank money settles on systems the banks do not operate: today a real-time gross settlement (RTGS) system such as Fedwire or T2, or a central bank ledger as in Project Agorá

### Problem 3: Binding the Ledger to the Core Banking System

The deposit stays in the issuing bank's core system while the claim circulates on the ledger. The two records must never promise the same funds twice, and the bank must show that claims in circulation match the liability it books, without exposing each customer's balance on the ledger.

**Requirements:**

- Mint only against funds on hold in the core system; release held funds only after the claim is burned
- That order leaves two possible interrupted states: funds held with no claim minted yet, and a claim burned with funds not yet released. Neither promises funds twice, and each has an owner and a time limit to complete or roll back
- Prove to the auditor and supervisor that the issuer's ledger supply equals its booked liability, without opening individual balances
- Mint and burn link to the bank's ISO 20022 notifications and payment instructions

**Constraints:**

- The core banking system stays the book of record; the ledger reconciles to it

## 5) Recommended Approaches

See [Approach: Private Payments](../approaches/approach-private-payments.md) for the shielded-transfer architectures that cover Problem 1. Deposit-specific considerations:

- Holder gating, issuer freeze and disclosure inside a shielded layer: [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md) and [Selective Disclosure](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- Interbank leg: [Private PvP via ERC-7573](../patterns/pattern-private-pvp-stablecoins-erc7573.md) (amount privacy only when the escrow contracts run inside a shielded layer) and the cross-network recovery paths in [Approach: Atomic DvP Settlement](../approaches/approach-dvp-atomic-settlement.md)
- Link to core banking: [Private ISO 20022 Messaging & Settlement](../patterns/pattern-private-iso20022.md)
- A bank operating its own ledger: [Approach: White-Label Infrastructure Deployment](../approaches/approach-white-label-deployment.md)

## 6) Open Questions

- Should each bank's tokenized deposit supply be public, visible to the operator, or provable to supervisors only?
- Can hold-and-mint and burn-and-release commit atomically across the core banking system and the ledger, or do banks accept interrupted states with an owner and a time limit?
- Who settles the interbank leg, and in what asset: RTGS, tokenized central bank reserves, or a settlement bank?
- How do mandated freezes and court-ordered reversals work inside a shielded layer?
- Can a deposit token be held by someone who is not a customer of the issuing bank, and does it remain a deposit if it is?

## 7) Notes And Links

- Related: [private-stablecoins.md](private-stablecoins.md) (non-bank issuer; same privacy patterns)
- Related: [private-payments.md](private-payments.md) (payment privacy patterns)
- Related: [private-fx.md](private-fx.md) (cross-border settlement; Swift tokenized-deposit pilot)
- Related: [private-treasuries.md](private-treasuries.md) (corporate cash on bank-operated ledgers)
- Jurisdictions: [EU / MiCA](../jurisdictions/eu-MiCA.md) · [US / SEC](../jurisdictions/us-SEC.md) (GENIUS Act)
- Standards: [ISO 20022](https://www.iso20022.org/) (messaging) · [ERC-7573](https://ercs.ethereum.org/ERCS/erc-7573) (Draft; conditional settlement, hides no amounts or counterparties)
