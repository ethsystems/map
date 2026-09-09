---
title: "Vendor: Renegade"
status: ready
maturity: production
---

# Renegade – On-chain Dark Pool (Private DEX with MPC + ZK)

## What it is

Renegade is a decentralized dark pool protocol for trading digital assets.  
It combines **maliciously-secure MPC** (for private order matching) with **zk-SNARK proofs** (for verifiable settlement), ensuring balances, orders, and trade history remain confidential while still settling on-chain.

## Fits with patterns

- [Private Shared State (co-SNARK)](../patterns/pattern-private-shared-state-cosnark.md)
- [co-SNARK](../patterns/pattern-co-snark.md)
- [Private ISO 20022](../patterns/pattern-private-iso20022.md)

## Not a substitute for

- Not a general L2 privacy rail (e.g. shielded stablecoin pools or confidential ERC-20).
- Not a compliance orchestration layer (no built-in regulator view keys).
- Not a settlement bridge (does not solve cross-chain atomicity directly).

## Architecture

- **Wallet model**: Each trader has a private wallet committed on-chain (`C(W) = H(B||O||F||K||r)`), with balances and orders hidden.
- **Key hierarchy**: Separate keys for root control, order matching, settlement, and viewing. Enables delegation without loss of custody.
- **Order matching**: Performed in MPC clusters (SPDZ-style protocols), ensuring neither relayers nor counterparties learn each other’s hidden state.
- **Settlement**: MPC outputs are wrapped in a collaborative zk-SNARK that proves correct matching. Settlement notes are encrypted under counterparties’ keys and appended to the global commitment tree.
- **Relayers**: Traders typically delegate matching to relayer clusters; relayers coordinate MPC handshakes but never see balances or order flow.

## Privacy domains

- **Balances**: hidden in wallet commitments.
- **Orders**: private until matched; no public order book.
- **Trade history**: only counterparties know their own trades.
- **Settlement**: encrypted notes prevent leakage of transfer details.

## Enterprise demand and use cases

- **Institutions / funds**: dark pool execution without revealing trading strategies.
- **Exchanges / brokers**: MEV-resistant order flow routing.
- **Large block traders**: reduced slippage via midpoint pricing in hidden pools.
- Early focus is on **crypto spot markets**, but design extends to derivatives or tokenized RWAs.

## Technical details

- Proof system: zk-SNARKs (Groth16-style collaborative proofs).
- MPC protocols: maliciously-secure SPDZ variants for order matching.
- Commitment scheme: Merkle trees for wallets and nullifiers.
- Encrypted settlement notes bound to on-chain commitments.
- Relayer clusters for scalability and redundancy.

## Strengths

- Eliminates MEV: no pre-trade or post-trade transparency.
- Fully private balances and orders, unlike lit AMMs.
- On-chain verifiability via zk-SNARK proofs.
- Delegation model allows separation of custody, matching, and settlement.

## Risks and open questions

- **Performance**: MPC order matching may be latency-sensitive (seconds–minutes).
- **Coordination**: relayer clusters add complexity and trust assumptions for liveness.
- **Regulatory access**: no built-in view-key system; compliance may require extensions.
- **Ecosystem fit**: interoperability with DeFi protocols still limited.

## Links

- [Renegade Whitepaper (2022)](https://renegade.fi/whitepaper.pdf)
- [Renegade Website](https://renegade.fi/)
- [GitHub – Renegade Finance](https://github.com/renegade-fi)
