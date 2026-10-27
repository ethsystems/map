---
title: "Pattern: Private Transaction Broadcasting"
status: ready
maturity: production
type: standard
layer: hybrid
last_reviewed: 2026-10-27

works-best-when:
  - Transaction content must stay hidden from the public mempool until block inclusion to prevent MEV extraction.
  - Large institutional trades must not signal intent to competitors before settlement.
  - A block-inclusion latency overhead of tens to low hundreds of milliseconds is acceptable.
avoid-when:
  - Public transparency of the pending transaction is required by policy or regulation.
  - Hard real-time inclusion guarantees are required and fallback to the public mempool is unacceptable.

context: both
context_differentiation:
  i2i: "Between institutions the relay or threshold committee can be contracted under SLA with diverse operators, audit logging, and an escalation path. Reputational and legal recourse bound the risk of a relay that peeks at or reorders transactions."
  i2u: "For end users the relay or threshold committee is operated by parties the user cannot audit. Protection strength depends on cryptographic enforcement (threshold decryption) rather than reputation, and on the availability of a fallback path that does not silently leak the transaction to the public mempool."

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Private relays can refuse individual transactions; censorship resistance depends on relay diversity and on a fallback path. Threshold-encrypted mempools reach `high` once a wide validator set enforces inclusion of encrypted payloads."
  o: "Core relay and threshold-decryption software is typically open source. Builder and relay operations often include proprietary orchestration and commercial MEV-capture logic."
  p: "Content is hidden from public mempool observers until inclusion. A compromised relay or a threshold committee above the threshold can reveal content early. Sender address and gas payer are typically visible even during the pending phase."
  s: "Relay-based variants rely on reputational and contractual enforcement. Threshold-encrypted variants rely on the honest-threshold assumption of the decryption committee and on timely decryption after inclusion."

post_quantum:
  risk: medium
  vector: "Threshold-encryption schemes built on pairings or elliptic-curve key agreement are broken by CRQC; recorded encrypted transactions face HNDL risk. Relay authentication channels typically use TLS with classical key exchange."
  mitigation: "Migrate threshold-encryption and relay authentication to post-quantum primitives (lattice-based KEMs and signatures) as the ecosystem ships them. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-threshold-encrypted-mempool, pattern-pretrade-privacy-encryption, pattern-focil-eip7805, pattern-mixnet-anonymity, pattern-onion-routing, pattern-shielding]
  see_also: [pattern-modular-privacy-stack, pattern-network-anonymity]

open_source_implementations:
  - url: https://github.com/flashbots/mev-boost
    description: "MEV-Boost relay infrastructure used by the majority of Ethereum validators"
    language: Go
  - url: https://github.com/shutter-network/rolling-shutter
    description: "Threshold-encrypted mempool reference stack (production on Gnosis Chain)"
    language: Go
---

## Intent

Hide transaction content from the public mempool before inclusion so that front-runners, sandwich bots, and competitors cannot observe or exploit pending trades. Transactions are routed through private relays or published as ciphertexts decrypted only after ordering, so adversaries see the transaction only once it is committed to a block.

## Components

- Private relay network that accepts signed transactions from users and forwards them to participating block builders under contractual or protocol rules.
- Builder set that includes privately-routed transactions in blocks without republishing them to the public mempool.
- Threshold decryption committee (for encrypted-mempool variants) that reveals transaction content only after the block containing it is committed.
- Client-side encryption library for threshold-encrypted submissions.
- RPC or wallet integration that routes traffic to the private endpoint with a configurable fallback policy.
- Optional MEV-redistribution mechanism that returns part of any extracted value to the originator.

## Protocol

1. [user] Sign the transaction locally with standard tooling.
2. [user] Submit the signed transaction to a private RPC endpoint instead of the public mempool.
3. [relayer] Forward the transaction to participating block builders under the relay's operating rules.
4. [validator] Include the transaction in a block without re-broadcasting to the public mempool.
5. [contract] Execute the transaction; its content becomes visible only after on-chain inclusion.
6. [relayer] Optionally return a share of extracted MEV to the originator via an MEV-sharing mechanism.

## Guarantees & threat model

Guarantees:

- Transaction content (target, value, calldata) is hidden from public mempool observers during the pending phase.
- Front-running, sandwiching, and just-in-time-liquidity attacks based on pre-inclusion observation are prevented.
- In the threshold-encrypted variant, ordering is committed before content is revealed, so ordering cannot depend on the plaintext.
- Execution semantics are unchanged from direct submission; the pattern is a transport change, not an execution change.

Threat model:

- In the relay variant: a colluding relay or builder can observe content; protection reduces to contractual and reputational enforcement. The relay or RPC endpoint also sees the submitter's IP address unless submission goes through a network anonymity layer.
- In the threshold variant: a committee above the decryption threshold can reveal content before inclusion, defeating the guarantee.
- Sender address and gas payer are typically visible during the pending phase.
- FOCIL inclusion lists are built from the public mempool, so once FOCIL ships (Hegotá) a privately routed transaction does not get its inclusion guarantee.
- A fallback to the public mempool on relay unavailability can silently leak the transaction; the fallback policy must be explicit.
- Post-inclusion transaction details are fully public; this pattern does not provide shielded amounts or counterparties.

## Trade-offs

- A privately routed transaction is included only when a builder it reached wins the slot. Three builders produced about 88% of Ethereum blocks in late September 2026 ([relayscan](https://www.relayscan.io/)), so the relay's builder coverage sets inclusion latency.
- Relay coverage varies over time; users should monitor live coverage metrics rather than assume a fixed figure.
- The threshold variant adds a key-release step after commitment, a decryption committee to operate, and a liveness dependency.
- Cost model varies: some relay services are free, others take a share of MEV or charge fees.

## Example

- A bank settles a large institutional stablecoin transfer by routing through a private relay so the transfer does not appear in any public mempool explorer. The block builder includes the transaction; competitors and MEV bots observe it only after confirmation.
- A dealer wins an RFQ and submits the settlement transaction encrypted under a threshold key. Validators commit the ciphertext to a block. Only after inclusion does the committee publish decryption shares, after which the transaction executes. Competing dealers cannot front-run or copy-trade the order.

## See also

- [Flashbots Protect documentation](https://docs.flashbots.net/flashbots-protect/overview)
- [Flashbots MEV-Share documentation](https://docs.flashbots.net/flashbots-mev-share/overview)
- [Shutter Network documentation](https://docs.shutter.network/)
- [MEV Blocker](https://mevblocker.io/)
