---
title: Private Bonds
status: ready
primary_domain: Funds & Assets
secondary_domain: Trading
---

## 1) Use Case

Institutional bond issuance and trading on public blockchains where volumes, prices, and positions must remain confidential to prevent front-running and strategy exposure. The solution requires confidential amounts and positions with selective regulator visibility, atomic delivery-versus-payment (DvP) settlement, while maintaining public-chain finality and economically viable daily settlement cycles.

## 2) Additional Context

## 3) Actors

Issuer · Investors · Crypto-Registry · Regulator · Oracles (valuation, payment) · Settlement venues (stablecoin network)

## 4) Problems

### Problem 1: Transaction Privacy with Regulatory Compliance

Public chains expose volumes, prices, and positions by default, enabling front-running and competitive intelligence gathering. Institutions require transaction-level confidentiality while maintaining selective disclosure for regulatory compliance.

**Requirements:**

- **Must hide:** amounts, positions, trade details (issuance + secondary); ideally RFQ/order size pre-inclusion
- **Public OK:** legal entity identities; existence of transactions/events (no figures)
- **Regulator access:** scoped viewing keys and/or ZK proofs; access logging ([attestations](../patterns/pattern-verifiable-attestation.md))
- **Settlement:** atomic DvP; minutes-level finality acceptable; **daily** cycles
- **Ops:** predictable **L2** costs (post-4844), append-only encrypted audit log with on-chain anchors; key rotation & retention policies

**Constraints:**

- Regulatory compliance (crypto-register integration where required)
- Production timeline: **1–2 years**
- Avoid HTLC brittleness
- Infrastructure costs viable during adoption phase

## 5) Recommended Approaches

See detailed solution architecture and trade-offs in [**Approach: Private Bonds**](../approaches/approach-private-bonds.md).

See [Approach: Atomic DvP Settlement](../approaches/approach-dvp-atomic-settlement.md) for the settlement leg, including cross-network DvP via ERC-7573.

**PoC Implementation:** [Private Bond PoC](https://github.com/ethsystems/pocs/tree/master/pocs/private-bond); three approaches: Custom UTXO (EVM), Privacy L2 (Aztec), and FHE (Zama fhEVM).

To keep orders and settlement transactions out of the public mempool before inclusion, see [Approach: Private Transaction Broadcasting](../approaches/approach-private-broadcasting.md).

## 6) Open Questions

_Notes below reference the PoC implementation._

- Secondary market: RFQ model + pre-trade privacy requirements (what must be hidden?)
  - _PoC approach:_ Negotiation and RFQ run off-chain with on-chain settlement; in Custom UTXO the issuer relays and can act as matcher, in Privacy L2 either party or a relayer submits the swap
- Minimum viable privacy: amounts/positions only vs parts of the term sheet?
  - _PoC approach:_ Amounts and positions hidden; legal entity identities can remain public (dual identity model)
- ISO 20022 relevance for bond workflows vs ICMA BDT usage

## 7) Notes And Links

- **Tracking:** Public workstream in [ethsystems/pocs / private-bond](https://github.com/ethsystems/pocs/tree/master/pocs/private-bond)
- **Standards:** [ERC-6123](https://ercs.ethereum.org/ERCS/erc-6123) (Draft; derivative/bond lifecycle), [ERC-7573](https://ercs.ethereum.org/ERCS/erc-7573) (Draft; conditional cross-chain DvP, atomicity rests on the decryption oracle, hides no amounts or counterparties), [ICMA Bond Data Taxonomy (BDT)](../patterns/pattern-icma-bdt-data-model.md)
- **Current standards:** ERC-20 tokens; HTLC sequences for cross-chain DvP (ERC-7573 is a proposed alternative); the approach targets same-chain atomic DvP, but no PoC implements the stablecoin leg: the Custom UTXO PoC's issuer-submitted `atomicSwap` is single-asset and the Privacy L2 PoC settles payment off-chain
- **Patterns:** [Atomic DvP via ERC-7573](../patterns/pattern-dvp-erc7573.md) · [Crypto-registry bridge (eWpG) with EAS mirroring](../patterns/pattern-crypto-registry-bridge-ewpg-eas.md)
- **Jurisdiction:** [DE / eWpG](../jurisdictions/de-eWpG.md) (crypto-securities register)
- **Domains:** [Funds & Assets](../domains/funds-assets.md) · [Trading](../domains/trading.md)
- Related: [private-corporate-bonds.md](private-corporate-bonds.md) (hiding funding signals from competitors)
- Related: [private-government-debt.md](private-government-debt.md) (auction integrity and holder privacy)
