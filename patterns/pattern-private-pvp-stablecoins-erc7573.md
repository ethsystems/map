---
title: "Pattern: Private PvP (cash to cash) Settlement via ERC-7573"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-09-11

works-best-when:
  - Two permissioned or regulated stablecoins (same L2 or cross-L2) must settle against each other with amount privacy, and both parties accept an oracle as the settlement trigger.
  - Finality sensing and pricing for FX or cross-issuer settlement can be served by an oracle feed both parties accept.
avoid-when:
  - Bilateral netting or off-chain wires already satisfy the settlement requirement.
  - HTLC timeouts with public amounts are acceptable and operational simplicity outweighs privacy.
  - The party that pays first cannot carry exposure to oracle failure after its payment settles.

context: i2i

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Censorship resistance depends on the oracle feed and the finality-relayer path. If the oracle or the cross-domain relayer censors, settlement can stall even though the escrow contracts are non-custodial."
  o: "The ERC-7573 escrow standard is open; oracle feeds and cross-chain messaging infrastructure are typically proprietary. Parties can substitute alternative oracles and relayers at deployment time."
  p: "Amounts and counterparties are hidden from the public chain, but the escrow contracts, oracle verdicts, and minimal settlement anchors remain visible. Metadata about when settlement happened and which assets are involved is not hidden."
  s: "Rides on the correctness of the escrow contracts and on the decryption oracle's honesty and liveness. Once the paying leg settles, an oracle or key-delivery failure leaves the payer exposed. Ops procedures must cover oracle outages and cross-domain message delays."

post_quantum:
  risk: medium
  vector: "Signatures over oracle reports, cross-domain proofs, and shielded-layer zero-knowledge proofs use elliptic-curve primitives that are broken by a CRQC."
  mitigation: "Migrate oracle attestations and cross-domain proofs to post-quantum signature schemes; compose with a post-quantum shielding layer on each leg. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-7573, ERC-20]

related_patterns:
  requires: [pattern-private-stablecoin-shielded-payments]
  composes_with: [pattern-dvp-erc7573, pattern-shielding, pattern-l2-encrypted-offchain-audit, pattern-regulatory-disclosure-keys-proofs]
  see_also: [pattern-cross-chain-privacy-bridge, pattern-pretrade-privacy-encryption]

open_source_implementations:
  - url: https://ercs.ethereum.org/ERCS/erc-7573
    description: "ERC-7573 specification for conditional-upon-transfer-decryption asset escrow"
    language: Solidity
---

## Intent

Settle two stablecoin legs against each other while keeping amounts and counterparties visible only to the transacting parties and their auditors. ERC-7573 makes the exchange conditional: one leg is locked and moves only on the verified outcome of the other, paying leg. A settled payment is final, so atomicity rests on the decryption oracle. Privacy comes from the shielded layers; ERC-7573 itself hides no amounts, identities, or timing. ERC-7573 defines no timeout: it replaces HTLC time-locks with trust in the oracle.

## Components

- ERC-7573 locking contract on the receiving side's chain: holds the locked leg under a trade identifier, bound to hashes of two outcome keys (release or return).
- ERC-7573 decryption contract on the paying side's chain: executes the payment and asks the decryption oracle (single-operator or threshold) for the key that matches the result.
- Shielded transfer layer on each chain that hides amounts and counterparties.
- Price or FX oracle feed, checked before the payment executes when the legs are in different units.
- Relayer or cross-domain messaging layer that carries the outcome key to the locking contract.
- Attestation layer for scoped disclosure to auditors, recording which parties settled, at what rate, and when.

## Protocol

1. [counterparty] Agree bilaterally on notional, FX rate (if cross-currency), oracle feed identifier, tolerance band, and trade identifier.
2. [counterparty] The receiving side locks its shielded stablecoin in the ERC-7573 locking contract, registering hashes of two outcome keys.
3. [counterparty] The paying side confirms the lock is final and the encrypted keys match the registered hashes for the right outcome and trade, then registers the trade terms and the encrypted keys in the ERC-7573 decryption contract on its chain.
4. [contract] The paying side pays through the decryption contract, which checks the amount and oracle rate against the terms, then settles or rejects the payment. A settled payment is final.
5. [operator] After the payment or cancellation outcome is final, the decryption oracle decrypts the success key if the payment settled, or the failure key if it was rejected or cancelled before payment.
6. [relayer] Submit the key to the locking contract. The success key releases the locked leg to the paying side; the failure key returns it to its owner.
7. [auditor] Use the attestation record and viewing keys on each leg to reconstruct the full settlement for compliance review.

## Guarantees & threat model

Guarantees:

- Conditional settlement across two chains or two assets: the locked leg moves only on the key for the paying leg's actual result. Assuming verified key setup, correct contracts, an honest and available oracle, finality on both chains, and eventual key delivery and inclusion, both legs settle or the locked leg returns.
- No cross-chain revert: if the success key is withheld after payment, the locked leg stays locked with no protocol exit, and the payer has already paid. Recovery is then bilateral or legal.
- Amount privacy: amounts are hidden on the chains themselves; only stakeholders and auditors with the viewing keys see the full trade.
- Scoped disclosure: attestations log regulator access without exposing amounts publicly.

Threat model:

- Decryption oracle honesty and liveness. A wrong key, or a withheld success key after payment, breaks atomicity against the paying side.
- Key setup. A key that does not match its registered hash strands the locked leg after payment. A success key known to the payer, or a failure key known to the locked leg's owner, before authorized release lets that party end up with both legs.
- Price oracle honesty and update cadence. A compromised or stale feed lets an attacker settle outside the agreed tolerance band.
- Finality on the paying leg. A success key released before payment finality lets a reorg reverse the payment after the locked leg has moved.
- Added deadlines. ERC-7573 has none. A reclaim deadline added by a deployment is safe only if key delivery is bounded; a late key lets the locked leg return after payment.
- Upgrade governance on both escrow contracts. A unilateral upgrade on one leg can freeze settled funds.
- Out of scope: correlation of settlement windows across the two chains by a global network observer.

## Trade-offs

- Oracle dependence is the main operational risk: outage, stale data, or price manipulation can each block settlement or produce an unfair release.
- Asymmetric exposure: the paying side carries the risk between payment and key delivery. Choose the paying side on that basis and cover the gap contractually.
- Cross-L2 finality sensing and the failure-recovery path add operational overhead relative to a single-chain DvP.
- Fragmentation across issuers and L2s may require market-making facilities to source liquidity on both sides.
- Debuggability: amounts and counterparties are hidden on-chain, so post-mortems require coordinated viewing-key access with the counterparty.

## Example

- Two regulated banks settle a shielded USD-stablecoin leg against a shielded EUR-stablecoin leg on separate L2s.
- The EUR-paying bank locks its leg in an ERC-7573 locking contract.
- The USD-paying bank pays through the decryption contract on its L2.
- Once the USD payment is final, the oracle releases the success key and a relayer submits it, releasing the EUR leg to the USD-paying bank.
- Chain observers see that the payment completed and the locked leg was released, but not the amounts or identities. The banks' auditors reconstruct the full trade via the shielded viewing keys and the attestation log.

## See also

- [ERC-7573: Conditional-upon-Transfer-Decryption for DvP](https://eips.ethereum.org/EIPS/eip-7573)
