---
title: "Domain: Trading"
status: draft
description: "Execute and settle without leaking intent, size, or counterparties to the wider market."
---

## TLDR
- Execution privacy: RFQ intent/size/price, pre-trade leakage control across private trading, derivatives, and OTC settlement.
- Public-sector and NGO procurement and settlement where counterparties, prices, or allocations are sensitive.
- Cross-domain atomicity (DvP/PvP) without HTLCs; optional zk-SPV for strong atomicity.

## Primary use cases
- [Private Repo](../use-cases/private-repo.md)
- [Private Stocks](../use-cases/private-stocks.md)
- [Private Commodities](../use-cases/private-commodities.md)

## Related use cases (secondary)
- [Private Bonds](../use-cases/private-bonds.md)

## Approaches
- [Approach: Atomic DvP Settlement](../approaches/approach-dvp-atomic-settlement.md)
- [Approach: Private Trade Settlement](../approaches/approach-private-trade-settlement.md)
- [Approach: Private Transaction Broadcasting](../approaches/approach-private-broadcasting.md)

## Shortest-path patterns
- [Pre-trade privacy (encrypted order flow)](../patterns/pattern-pretrade-privacy-encryption.md)
- [Threshold-encrypted mempool](../patterns/pattern-threshold-encrypted-mempool.md)
- [Atomic DvP via ERC-7573](../patterns/pattern-dvp-erc7573.md)
- [Shielding](../patterns/pattern-shielding.md)
- [Commit-and-Prove fallback](../patterns/pattern-commit-and-prove.md)

## Adjacent vendors
- [Renegade](../vendors/renegade.md)
- [Aztec L2](../vendors/aztec.md)
- [Railgun](../vendors/railgun.md)
- [Flashbots](../vendors/flashbots.md)
- [Shutter](../vendors/shutter.md)
- [Hinkal](../vendors/hinkal.md)
- [Orion Finance](../vendors/orion-finance.md)
