---
title: "Vendor: Flashbots"
status: ready
maturity: production
---

# Flashbots – MEV Infrastructure & SUAVE

## What it is

Flashbots develops MEV infrastructure including private mempools, block building, and SUAVE (Single Unifying Auction for Value Expression), a decentralized platform for private intent expression and execution. Flashbots Auction is in production on Ethereum mainnet; SUAVE is under development.

## Fits with patterns

- [Pre-trade Privacy](../patterns/pattern-pretrade-privacy-encryption.md): private intent expression via SUAVE
- [Private Broadcasting](../approaches/approach-private-broadcasting.md): MEV protection through private mempools

## Not a substitute for

- Complete MEV elimination (redistributes rather than eliminates)
- Privacy after execution (transactions remain visible on-chain)
- Regulatory compliance frameworks
- Long-term privacy guarantees (relies on builder trust)

## Architecture

**Flashbots Auction:** transactions are submitted directly to builders, bypassing the public mempool; MEV-Share lets users capture a portion of the MEV from their transactions; competitive block construction handles MEV extraction.

**SUAVE (under development):** a universal intent pool for cross-domain intent expression and matching, programmable privacy (selective revelation of intent information), and decentralized execution across multiple environments.

## Enterprise demand and use cases

- Private transaction submission bypassing the public mempool
- MEV protection through direct builder relationships
- Revenue sharing via MEV-Share
- Cross-chain intent expression (SUAVE)

## Technical details

- **Networks:** Ethereum mainnet (Flashbots Auction); multi-chain (SUAVE)
- **Developer tools:** MEV-Share SDK, Flashbots Auction APIs
- **Builder network:** requires integration with participating builders
- **Intent standards:** SUAVE is developing cross-chain intent standards

## Strengths

- Established MEV mitigation with significant adoption
- Strong relationship with the Ethereum ecosystem
- Active development of new infrastructure (SUAVE)
- Revenue-sharing mechanisms align user incentives

## Risks and open questions

- Centralization concerns around builder relationships
- SUAVE still in development
- Limited privacy guarantees (trust-based model)
- Regulatory uncertainty around MEV redistribution

## Links

- [Flashbots](https://flashbots.net/)
- [MEV-Share](https://docs.flashbots.net/flashbots-mev-share/overview)
- [SUAVE](https://writings.flashbots.net/the-future-of-mev-is-suave)
- [Documentation](https://docs.flashbots.net/)
