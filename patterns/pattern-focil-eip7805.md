---
title: "Pattern: FOCIL - Fork-Choice Enforced Inclusion Lists"
status: ready
maturity: testnet
type: standard
layer: L1
last_reviewed: 2026-10-27

works-best-when:
  - Censorship resistance is critical and must not depend on builder honesty.
  - Block builders concentrate power and may selectively censor transactions.
  - Same-slot inclusion guarantees are required (no one-slot delay).
avoid-when:
  - Mempool visibility of transaction details is unacceptable (combine with encrypted mempools instead).

context: both
context_differentiation:
  i2i: "Between institutions, inclusion guarantees protect bilateral settlement against builder-level censorship, but trade details remain publicly visible in the mempool. Acceptable where counterparties already publish identities and amounts; inadequate where institutional trades must stay confidential from competitors. Pair with encrypted-mempool patterns to cover trade details between institutional counterparties."
  i2u: "The user is asymmetric to block builders and cannot negotiate inclusion directly. The 1-of-16 committee honesty assumption provides protocol-level protection against OFAC-style censorship or MEV-motivated exclusion. Public transaction details still expose end-user activity to front-running and surveillance; encrypted submission is a priority for user-facing deployments."

crops_profile:
  cr: high
  o: yes
  p: none
  s: high

crops_context:
  cr: "Reaches `high` through a 1-of-16 committee honesty assumption: a single honest IL committee member suffices to force-include a transaction. Drops if the P2P gossip network is partitioned or if committee selection is subverted."
  o: "Fully open specification (EIP-7805) implemented in consensus layer clients. Any staked validator rotates through the committee, and any node can verify IL satisfaction."
  p: "No privacy: transaction details remain fully visible in the mempool and on chain. Would reach `partial` if inclusion lists carried sealed transactions, as LUCID (EIP-8184, Draft) proposes."
  s: "Inherits the security of Ethereum consensus. Fork-choice enforcement means non-compliant blocks cannot become canonical, so the guarantee is as strong as the underlying finality mechanism."

post_quantum:
  risk: medium
  vector: "BLS signatures used for committee IL signatures and attestations are broken by a CRQC. HNDL risk is low since IL contents are public."
  mitigation: "Migrate consensus signatures to a post-quantum scheme (hash-based or lattice-based) as part of the broader Ethereum consensus PQ transition."

standards: [EIP-7805]

related_patterns:
  composes_with: [pattern-pretrade-privacy-encryption, pattern-threshold-encrypted-mempool, pattern-private-stablecoin-shielded-payments]
  alternative_to: [pattern-private-transaction-broadcasting]
  see_also: [pattern-verifiable-attestation, pattern-privacy-l2s]

open_source_implementations:
  - url: https://eips.ethereum.org/EIPS/eip-7805
    description: "EIP-7805 specification and reference implementation notes."
    language: Specification
---

## Intent

Prevent censorship of Ethereum transactions by sophisticated block builders. A committee of 16 validators per slot can force-include transactions in the next block; non-compliant blocks fail fork-choice and cannot reach finality. The pattern provides no privacy on its own: transactions remain fully visible in the mempool and on chain. It guarantees only that a transaction cannot be censored once it reaches the public mempool. FOCIL is scheduled for inclusion in the Hegotá upgrade, estimated for mid-2027.

## Components

- IL committee of 16 validators selected per slot to construct inclusion lists for the following slot.
- Inclusion list is a signed, bounded set of transactions (8 KiB max per member) that the next block must include.
- P2P gossip network propagates signed inclusion lists across the validator set.
- Fork-choice enforcement is the consensus-layer rule: attesters refuse to attest to blocks that fail to satisfy observed ILs.
- Equivocation detection identifies committee members publishing conflicting ILs and discards their contributions.

## Protocol

1. [consensus] A committee of 16 validators is selected for slot N to build inclusion lists for slot N+1.
2. [committee] Between t=0s and t=8s of slot N, committee members monitor the public mempool and assemble inclusion lists of transactions they want force-included.
3. [committee] At t=8s, committee members broadcast signed inclusion lists over the P2P network after processing slot N's block.
4. [validator] At t=9s, validators freeze their view of inclusion lists; later ILs are rejected, and equivocators are flagged.
5. [builder] At t=11s, the builder for slot N+1 collects observed inclusion lists and constructs an execution payload that includes their transactions anywhere in the block.
6. [proposer] At t=0s of slot N+1, the proposer broadcasts the block containing the IL transactions.
7. [attester] At t=4s, attesters verify that every non-equivocated IL transaction is included or provably invalid; otherwise they withhold attestations.
8. [consensus] Fork-choice rejects any block that fails IL satisfaction, so a non-compliant block cannot become canonical.

## Guarantees & threat model

Guarantees:

- Censorship resistance under builder centralization via a 1-of-16 committee honesty assumption.
- Same-slot inclusion: constraints for slot N+1 include transactions visible in slot N (no one-slot delay).
- Builder flexibility: the builder chooses placement of IL transactions within the block, reducing incentives for side channels.
- Fork-choice enforced: non-compliant blocks receive no attestations and cannot finalize.

Threat model:

- At least one honest and well-connected IL committee member per slot; the P2P network reliably delivers their list before the view-freeze deadline.
- Non-malicious builder with sufficient bandwidth between view freeze (t=9s) and block broadcast; builders disconnected from the committee can miss IL transactions.
- No privacy protection. Transaction details remain publicly visible throughout the process.
- Out of scope: transactions sent privately to builders never reach the public mempool, so inclusion lists do not cover them. Until sealed transactions can be listed (LUCID), a sender chooses between pre-inclusion privacy and FOCIL's inclusion guarantee.

## Trade-offs

- Additional bandwidth: up to 16 × 8 KiB inclusion-list gossip per slot, plus potential O(n^2) validity checks if naively implemented.
- Liveness risk: insufficient time between view freeze and block broadcast can cause builders to miss ILs, especially for poorly-connected builders.
- Equivocation handling: the P2P rule allows forwarding up to two inclusion lists per committee member, so bandwidth can double in the worst case.
- No explicit incentives: the pattern relies on altruistic behavior; there are no direct rewards for IL committee members.
- Complexity: requires coordinated changes across consensus-layer fork-choice rules, execution-layer validation, and the P2P network.

## Example

A bank submits a €5M stablecoin transfer to the public mempool at t=0s of slot N. Transaction details are fully visible to all observers. An IL committee member includes it in their inclusion list and broadcasts at t=8s. At t=9s the view freezes. The slot N+1 builder collects the IL at t=11s and includes the transfer in the block. Attesters verify inclusion at t=4s. Even if the builder would have preferred to censor the payment (for OFAC compliance, MEV extraction, or competitive intelligence), fork-choice enforcement makes censorship non-viable. Privacy impact is zero: the €5M amount, counterparty, and timing remain publicly visible throughout.

## See also

- [EIP-7547: forward inclusion lists](https://eips.ethereum.org/EIPS/eip-7547) for a one-slot-delay alternative
- [Private Payments Approach](../approaches/approach-private-payments.md)
- [EIP-8184: LUCID encrypted mempool](https://eips.ethereum.org/EIPS/eip-8184), which extends inclusion lists to sealed transactions
- [Private Transaction Broadcasting Approach](../approaches/approach-private-broadcasting.md)
