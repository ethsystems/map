---
title: "Vendor: Fairblock"
status: ready
maturity: testnet
---

# Fairblock: Cross-chain Confidential Transfers (lightweight homomorphic ops + ZK)

## What it is

Fairblock provides confidential balances and transfers for assets (e.g., stablecoins) that remain issued and liquid on an originating chain (e.g., EVM networks), while confidentiality logic executes on **FairyRing**, a purpose-built confidentiality execution layer.

The core idea is a cross-chain confidential layer: users lock tokens into a minimal smart contract on the originating chain, receive an encrypted balance maintained by FairyRing, and execute confidential transfers backed by zero-knowledge proofs and homomorphic balance updates, then optionally withdraw back to the originating chain.

## Fits with patterns

- [Selective disclosure (viewing keys + zero-knowledge proofs)](../patterns/pattern-regulatory-disclosure-keys-proofs.md): scoped disclosure for audit and compliance workflows

## Not a substitute for

- Anonymity, mixers, or large anonymity sets: Fairblock focuses on amount and balance confidentiality, not obfuscasting participant identities by default.
- Stealth addresses (unlinkable recipients): requires separate stealth address infrastructure if counterparty unlinkability is required.
- Full transaction graph privacy: relationship metadata may still be inferable depending on integration design.

## Architecture

High-level components:

1. Origin chain (e.g., EVM): Minimal locking smart contract
   - Locks/unlocks the underlying ERC-20 tokens
   - User-facing interface for deposit/transfer/withdraw flows
2. FairyRing: confidentiality execution layer
   - CosmWasm contracts maintain an encrypted chain
   - Verifies zero-knowledge proofs for valid encrypted state transitions
   - Performs lightweight homomorphic add/subtract for balance updates
3. Cross-chain messaging (IBC-style)
   - Relays packets between the origin chain and FairyRing
   - Relayers transport packets but are not intended to be trusted for correctness

## Privacy domains

Encrypted (intended):

- Amounts and balances (encrypted state)

Public (intended):

- Sender and receiver addresses (confidentiality, not anonymity)
- Existence and timing of origin-chain transactions that invoke the locking contract

## Selective disclosure (compliance posture)

FairyRing supports selective disclosure for audits and investigations via scoped decryption access using threshold identity-based encryption (IBE), aiming to avoid a single persistent global audit key

## Enterprise demand and use cases

- Treasury operations: encrypt balances and flows while keeping counterparties known and auditable
- Payroll and vendor payouts: amount confidentiality with the ability to disclose specific transactions when required
- B2B payment rails: confidentiality layer for regulated payment operations that still require reporting and audit

## Technical details

- Cross-chain design connecting an origin chain locking contract with FairyRing as confidentiality layer
- Encrypted layer and zero-knowledge proof verification for valid state transitions
- Lightweight homomorphic operations for balance updates
- Threshold IBE for scoped selective disclosure

## Strengths

- Preserves existing asset issuance and liquidity on EVM chains (no token contract changes required)
- Clear compliance posture via scoped selective disclosure (IBE-based); avoids a single persistent audit key
- Security: avoids off-chain coprocessing, single-TEE trust models, MPC honest-majority assumptions, and external prover service dependencies (proof generation is local)
- Performance/UX: no bridging of funds and no new wallet behavior; lightweight cryptography targets low latency flows (vs. long proof delays common in many privacy systems)

## Risks and open questions

- Relationship privacy is optional: the default model prioritizes confidentiality (amounts/balances) with traceable addresses; deployments that need relationship obfuscation/anonymity can layer additional primitives, but that typically increases complexity/latency and may introduce stricter compliance requirements that not all partners want
- Cross-chain messaging (liveness, not custody): messaging mainly impacts availabiilty and UX (e.g., delayed credits or withdrawals). Funds remain locked on the original chain; messaging failtures should not imply loss of funds, but operational reliability and integration monitoring still matter
- Locking smart contract: the origin-chain locking contract is a critical component
- Disclosure governance: “who can request or approve disclosure, under what policy, and how it’s audited” must be specified in deployments

## Links

- Fairblock docs (confidential transfers): https://docs.fairblock.network/docs/confidential_transfers/confidential_transactions
- Technical overview (architecture): https://docs.fairblock.network/docs/confidential_transfers/technical_overview
- Stabletrust (UI / UX layer): https://docs.fairblock.network/docs/confidential_transfers/stabletrust
- FairyRing repo: https://github.com/Fairblock/fairyring
