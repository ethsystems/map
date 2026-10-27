---
title: "Vendor: Flashbots"
status: ready
maturity: production
---

# Flashbots – MEV Infrastructure

## What it is

Flashbots develops MEV infrastructure for Ethereum: MEV-Boost, which validators use to source blocks from builders; Flashbots Protect, a private RPC that keeps transactions out of the public mempool; MEV-Share, which lets users share hints with searchers and capture part of the MEV their transactions create; and BuilderNet, a block-building network run by several operators. SUAVE, its earlier design for a decentralized intent platform, is archived.

## Fits with patterns

- [Private Transaction Broadcasting](../patterns/pattern-private-transaction-broadcasting.md): Protect is a private relay
- [Pre-trade Privacy](../patterns/pattern-pretrade-privacy-encryption.md): private order flow through Protect and MEV-Share
- [Private Broadcasting](../approaches/approach-private-broadcasting.md): the Private Relay sub-approach
- [TEE-Based Privacy](../patterns/pattern-tee-based-privacy.md): BuilderNet's services that run in TEEs

## Not a substitute for

- Complete MEV elimination (redistributes rather than eliminates)
- Privacy after execution (transactions remain visible on-chain)
- Regulatory compliance frameworks
- Long-term privacy guarantees (relies on builder trust)

## Architecture

**MEV-Boost:** open-source middleware for proposer-builder separation. Validators run it to take the most profitable block offered by builders through relays, which hold each block's contents until the proposer signs its header. Flashbots operates one of the relays. A privately sent transaction needs a builder that received it to win the block. ePBS (EIP-7732, scheduled for Glamsterdam) moves proposer-builder separation into the protocol.

**Flashbots Auction:** searchers and users send bundles of transactions directly to block builders with the `eth_sendBundle` RPC, bypassing the public mempool. It runs on MEV-Boost. The documentation now files it under "Flashbots Builders", though its pages and URLs still call it Flashbots Auction.

**Flashbots Protect:** a private RPC. By default it sends transactions only to the Flashbots builder. Users can add builders, which means trusting those builders too; a fast mode shares transactions with all registered builders.

**MEV-Share:** an operator-run node shares configurable hints about pending transactions with searchers, who may only backrun them. In Protect's fast mode, searchers running in TEEs receive the full transaction whatever the hint settings.

**BuilderNet:** a block-building network operated by several parties. Since September 2026 its documentation says it is "transitioning to a new security model": during this period its nodes do not run in TEEs, while its backrunning and routing services still do.

**SUAVE (archived):** a planned decentralized intent and block-building platform. The suave-geth repository was archived in May 2025.

## Privacy domains

- **Bundle submission (Auction):** bundles reach the builders the sender chooses and skip the public mempool.
- **Private submission (Protect):** pending transactions are hidden from the public mempool, not from the builders that receive them.
- **Hint sharing (MEV-Share):** the user chooses which transaction fields searchers see before inclusion.

## Enterprise demand and use cases

- Private transaction submission bypassing the public mempool
- MEV protection through direct builder relationships
- Revenue sharing via MEV-Share

## Technical details

- **Networks:** Ethereum mainnet
- **Developer tools:** Protect RPC, MEV-Share SDK, bundle APIs
- **Code:** [mev-boost](https://github.com/flashbots/mev-boost) (Go), [rbuilder](https://github.com/flashbots/rbuilder) (Rust)

## Strengths

- MEV-Boost is the block-sourcing path for most Ethereum validators
- Protect needs only an RPC change in the wallet or submission stack
- Revenue-sharing mechanisms return part of the MEV to users
- Open-source relay and builder software

## Risks and open questions

- Trust-based privacy: the builders a transaction reaches can read it
- Inclusion depends on the share of blocks won by the builders a transaction reaches
- BuilderNet's TEE-based security model is in transition (September 2026)
- Regulatory uncertainty around MEV redistribution

## Links

- [Flashbots](https://flashbots.net/)
- [Flashbots Protect settings](https://docs.flashbots.net/flashbots-protect/settings-guide)
- [MEV-Share](https://docs.flashbots.net/flashbots-mev-share/introduction)
- [BuilderNet documentation](https://buildernet.org/docs/)
- [Documentation](https://docs.flashbots.net/)
