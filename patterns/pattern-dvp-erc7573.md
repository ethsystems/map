---
title: "Pattern: Atomic DvP via ERC-7573 (cross-network)"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-09-11

works-best-when:
  - Asset and cash legs live on different networks (L1 or L2).
avoid-when:
  - Both legs already settle on the same network with simple on-chain transfers.
  - You're searching for a production-ready solution (EIP not yet recommended).
  - You're searching for a fully trustless solution.

context: i2i

crops_profile:
  cr: medium
  o: partial
  p: none
  s: medium

crops_context:
  cr: "Oracle governance is typically codified in bilateral or consortium agreements, so the practical censorship risk sits below the protocol-level score. Reaches `high` if threshold decryption oracles with k-of-n quorum replace single-operator oracles, removing any single exclusion point."
  o: "Oracle and contract implementations vary by deployment. Improves to `yes` by releasing oracle implementations and outcome key services under a copyleft license."
  p: "Standard ERC-7573 provides atomic settlement but no privacy: trade identifier, amounts, and counterparties may be observable on-chain. Reaches `partial` or `full` by layering threshold-encrypted mempools, minimal on-chain trade data, and private or proof-based payment layers."
  s: "Rides on the decryption oracle operator's honesty for liveness. Improves to `high` by replacing operational trust with verifiable proofs of outcome key validity and threshold quorum."

post_quantum:
  risk: medium
  vector: "Oracle decryption relies on standard public-key encryption; outcome key commitments inherit hash assumptions. Payment-network signatures carry host-chain PQ exposure."
  mitigation: "Migrate decryption to PQ-safe KEM schemes (Kyber, ML-KEM) as ERC-7573 extensions mature. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-7573, ERC-20]

related_patterns:
  composes_with: [pattern-mpc-custody, pattern-regulatory-disclosure-keys-proofs, pattern-threshold-encrypted-mempool]
  alternative_to: [pattern-commit-and-prove]
  see_also: [pattern-private-pvp-stablecoins-erc7573]

open_source_implementations: []
---

## Intent

Enable atomic Delivery-versus-Payment across two networks so that either the asset is delivered and payment is completed, or the asset returns to the seller. The pattern targets institutions that need cross-network settlement with predictable failure behavior, auditability, and optional privacy extensions layered on top.

In this pattern:

- asset leg is the tokenized security or collateral being delivered.
- cash leg is the token used for payment, such as a stablecoin or tokenized deposit.
- decryption oracle is a service on the payment network that, for each trade, decrypts one of two outcome keys based on the actual payment result; the asset contract then uses that key to decide whether to deliver or reclaim.

## Components

- Locking contract on the asset network registers the hashed outcome keys per trade identifier and gates delivery of the locked asset.
- Decryption contract on the payment network holds the encrypted outcome keys and calls the oracle after observing the payment result.
- Decryption oracle is a single or multi-operator service that decrypts the outcome key matching the actual payment result. Stateless by design.
- Off-chain trade negotiation layer assigns a shared trade identifier, creates and distributes two outcome keys per trade (success and failure), and holds them off-chain.
- Oracle proxy and callback contracts bridge the decryption contract and the oracle per the ERC-7573 interface.
- Monitoring and logging infrastructure watches both networks for event correlation and reconciles trade state.

## Protocol

1. [user] Two institutions agree off-chain on the asset, quantity, payment token, amount, shared trade identifier `T`, and a target settlement time (ERC-7573 enforces no on-chain deadline).
2. [user] The trade-setup system generates two outcome keys for `T` (one meaning "deliver to buyer", one meaning "return to seller") and distributes them off-chain.
3. [contract] The seller locks the asset in the locking contract on the asset network under `T`, registering hashed values of the two outcome keys.
4. [contract] The buyer registers `T` and payment details in the decryption contract on the payment network, along with encrypted forms of the same two outcome keys.
5. [user] The buyer executes the payment through the decryption contract. The contract checks the payment against the registered details for `T`.
6. [operator] The decryption contract calls the oracle, which decrypts the encrypted key matching the actual outcome (success or failure) and returns it only after that outcome is final.
7. [contract] An authorized party submits the outcome key to the locking contract, which verifies it against the registered hashes and either delivers the asset to the buyer (on success) or lets the seller reclaim it (on failure).

## Guarantees & threat model

Guarantees:

- Atomic settlement, conditional on verified key setup, correct contracts, an honest and available decryption oracle, finality on both networks, and eventual key delivery and inclusion: for each trade, both asset and cash legs settle, or the payment fails and the seller reclaims the asset. A completed payment is final; the protocol cannot reverse it.
- Defined failure behavior: if the payment fails or is cancelled, the failure key lets the seller reclaim the asset. ERC-7573 defines no timeout. A latest-time reclaim added by a deployment is safe only if key delivery is bounded; a late key lets the seller reclaim after the buyer has paid.
- Optional privacy extensions allow observers to see that a trade settled or not without seeing full terms; institutions can still disclose details off-chain when required.

Threat model:

- Soundness of the commitment scheme binding outcome key hashes.
- Non-colluding decryption oracle operators. A single operator in a centralized deployment can withhold decryption or release the wrong outcome key, breaking atomicity or liveness.
- Non-censoring sequencers on both networks during the settlement window. A censored payment or key-submission transaction blocks settlement or reclaim until it is included.
- Honest trade-setup system that generates unique, unpredictable outcome keys and distributes them correctly. Collision or replay across trades breaks the guarantee.
- Network-layer metadata (IP, timing, gas patterns) is out of scope.

## Trade-offs

- Oracle governance is a recurring operational question. Institutions must agree on who runs the oracle, how its keys are managed, and how incidents (downtime, misconfiguration) are handled.
- Settlement latency depends on payment finality, oracle processing, and any proof generation, adding delay relative to same-network transfers.
- Integration effort is non-trivial. Internal systems must handle outcome keys, track trade identifiers, listen to events on both networks, and reconcile them with internal ledgers.
- Exceptional cases (incorrect parameters, operational errors, regulatory holds) still require documented off-chain procedures. Fully automated recovery is out of scope.
- Standard ERC-7573 provides no privacy. Confidentiality requires layering additional patterns: threshold oracles, minimal on-chain data, or private payment layers.

## Example

- An issuer issues a tokenized bond on Ethereum L1 (asset leg). A buyer holds EURC stablecoin on an L2 rollup (cash leg).
- They agree off-chain on trade identifier `T`, bond quantity, payment amount, and a target settlement time.
- The seller locks the bond in the locking contract on L1 under `T`.
- The buyer registers `T` and executes the EURC payment via the decryption contract on the rollup; the oracle releases the success outcome key for `T`.
- That key is submitted to the L1 locking contract, which transfers the bond to the buyer. If the payment had failed or been cancelled, the failure outcome key would have been used instead and the seller would reclaim the bond.

## Privacy extensions

Standard ERC-7573 provides atomic settlement but no privacy. For institutional use cases requiring confidentiality:

- Threshold decryption oracle: Run the oracle with several independent operators and require a k-of-n quorum before an outcome key is released.
- Minimal on-chain trade data: Keep full trade terms (price, size, counterparties) in internal systems; on-chain, store only a trade identifier and a short reference that links back to those records.
- Private or proof-based payment layer: Use a network or rollup for the cash leg that hides detailed balances but can provide a clear "payment completed or not completed" result for each trade identifier.

These extensions do not change how the contracts decide outcomes: the asset contract still receives an outcome key and either delivers the asset or allows reclaim.

## See also

- [ERC-7573 specification](https://ercs.ethereum.org/ERCS/erc-7573)
- [Private Trade Settlement approach](../approaches/approach-private-trade-settlement.md)
