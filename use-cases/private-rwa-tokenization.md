---
title: Private RWA Tokenization
status: ready
primary_domain: Funds & Assets
secondary_domain:
---

## 1) Use Case

Regulated real-world assets (RWAs) are tokenized on-chain to enable permissioned transfers of ownership and co-ownership. The solution requires private pricing and valuation with full history, unlinkable ownership and transfer histories, exportable ownership with accreditation proofs, and policy enforcement based on attestations with regulatory intervention capabilities.

## 2) Additional Context


### Market Signals

- **Market size:** ~$38.6B distributed on-chain RWA value, excluding stablecoins (29 Sep 2026, [rwa.xyz](https://app.rwa.xyz/)); rwa.xyz counts a further ~$358B of [represented assets](https://rwa.xyz/blog/a-new-framework-for-tokenized-assets-distributed-and-represented), which are recorded on-chain but cannot leave the issuing platform or move peer-to-peer. [Binance Research](https://www.binance.com/en/research/analysis/the-rwa-activation-era), using [DefiLlama](https://defillama.com/rwa) data, reports $34.18B (15 Sep 2026)
- **Growth:** [Coinbase](https://assets.ctfassets.net/sygt3q11s4a9/6oinXHvVekdIUw2Ch7yIQw/22d0185eba3c49322ce7cf0d287ea872/SOCQ2Report_final.pdf) reported ~245× growth, $85M (April 2020) to $21B (April 2025), using RWA.xyz data under its methodology at the time; not directly comparable to the figures above
- **Asset distribution:** Share of distributed value (29 Sep 2026, rwa.xyz): US Treasuries (38%), credit (20%), commodities (13%), active strategies (10%), stocks (8%), private equity / venture capital (6%), non-US government debt (3%), real estate (<1%)
- **Deployed categories:** [US Treasuries](https://app.rwa.xyz/treasuries), [Non-US Government Debt](https://app.rwa.xyz/government-bonds), [Credit](https://app.rwa.xyz/credit), [Commodities](https://app.rwa.xyz/commodities), [Active Strategies](https://app.rwa.xyz/active-strategies), [Private Equity / VC](https://app.rwa.xyz/private-equity-venture-capital), [Real Estate](https://app.rwa.xyz/real-estate), [Stocks](https://app.rwa.xyz/stocks)

## 3) Actors

Identity Provider (onchain identities for issuers, investors, authorized regulators) · Oracle (compliance checks) · Issuer (financial institutions, authorized registrar) · Investor · Regulator (policy enforcement, intervention capabilities)

## 4) Problems

### Problem 1: Privacy-Preserving RWA Tokenization with Regulatory Compliance

On-chain RWA tokenization exposes ownership, valuation, and transfer history by default, conflicting with institutional privacy requirements and competitive positioning. The solution must provide ownership privacy with unlinkable transfer histories while maintaining selective disclosure and enforcement capabilities.

**Requirements:**

- **Must hide:** who owns the RWA (and amount if not NFT), history of ownership, value if NFT
- **Public OK:** asset existence; contract code; compliance schema; [attestation](../patterns/pattern-verifiable-attestation.md) framework
- **Regulator access:** selective disclosure of trade info such as owner id, amount/value of RWA, and pause/freeze RWA (granularity varies by jurisdiction)
- **Settlement:** atomic DvP
- **Ops:** N/A

**Constraints:**

- Cannot break existing regulatory frameworks
- Must support authorized identity issuance and management
- Must provide auditability and accountability for [attestations](../patterns/pattern-verifiable-attestation.md) and issuer solvency

## 5) Recommended Approaches

See [Private Bonds](../approaches/approach-private-bonds.md) and [Private Payments](../approaches/approach-private-payments.md) approaches - RWA tokenization can inherit private transfer solutions with emphasis on transfer compliance checks and RWA-specific standards (ERC-3643, ERC-7943, or CMTAT with privacy modifications using commitments).

## 6) Open Questions

- N/A

## 7) Notes And Links

- **Patterns:**
  - [pattern-private-pvp-stablecoins-erc7573.md](../patterns/pattern-private-pvp-stablecoins-erc7573.md)
  - [pattern-erc3643-rwa.md](../patterns/pattern-erc3643-rwa.md)
  - [pattern-l2-encrypted-offchain-audit.md](../patterns/pattern-l2-encrypted-offchain-audit.md)
  - [pattern-zk-kyc-ml-id-erc734-735.md](../patterns/pattern-zk-kyc-ml-id-erc734-735.md)
  - [pattern-co-snark.md](../patterns/pattern-co-snark.md)
- **Standards:** ERC-3643, ERC-7943, CMTAT
