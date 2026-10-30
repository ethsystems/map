---
title: "Approach: Private Transaction Broadcasting"
status: ready
last_reviewed: 2026-10-27

use_case: private-stocks
related_use_cases: [private-derivatives, private-fx, private-bonds, private-stablecoins]

primary_patterns:
  - pattern-private-transaction-broadcasting
  - pattern-threshold-encrypted-mempool
  - pattern-pretrade-privacy-encryption
  - pattern-privacy-l2s
supporting_patterns:
  - pattern-shielding
  - pattern-network-anonymity
  - pattern-focil-eip7805

open_source_implementations:
  - url: https://github.com/flashbots/suave-geth
    description: "SUAVE private-intent execution (suave-geth archived May 2025; Flashbots migrated to BuilderNet)"
    language: Go
  - url: https://github.com/shutter-network/rolling-shutter
    description: "Shutter Network keyper (threshold-encrypted mempool, production on Gnosis Chain)"
    language: Go
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec privacy-native L2"
    language: TypeScript / Noir
---

# Approach: Private Transaction Broadcasting

## Problem framing

### Scenario

A trading desk routes large institutional orders against on-chain venues. Visible transactions in the public mempool enable front-running, sandwiching, and intent inference; the spread cost on a EUR 25M order is large enough to dominate fees for the year. The desk needs broadcasting infrastructure that hides the order content, the timing, or both, while preserving validity and ordering guarantees. An issuer faces the same exposure with compliance actions: a pending freeze or blocklist update in the public mempool lets the affected holder move funds first with a higher priority fee.

### Requirements

- Hide transaction content (order side, size, target venue) before inclusion
- Mitigate MEV extraction (front-running, sandwiching, back-running)
- Preserve transaction validity and ordering guarantees
- Maintain audit trail for compliance and post-trade reporting
- Integrate with existing custody and execution workflows

### Constraints

- Public mempool is the default; opting out incurs operational complexity
- Sequencer or builder trust is the typical price of privacy at this layer
- Cross-network coordination amplifies MEV exposure unless each leg is privatized
- On Ethereum L1 no encrypted mempool is live, and FOCIL's inclusion guarantee (Hegotá) covers only the public mempool

## Approaches

### Private Relay

```yaml
maturity: production
context: both
crops: { cr: low, o: partial, p: partial, s: medium }
uses_patterns: [pattern-private-transaction-broadcasting, pattern-network-anonymity]
example_vendors: [flashbots]
```

**Summary:** Send the signed transaction to a private RPC endpoint or straight to contracted builders instead of the public mempool. Content stays hidden from mempool observers, not from the relay or the builders.

**How it works:** The wallet or custody stack submits to a private endpoint ([Flashbots](../vendors/flashbots.md) Protect, MEV Blocker) or to builders under bilateral agreements. Builders include the transaction without rebroadcasting it. See [Private Transaction Broadcasting](../patterns/pattern-private-transaction-broadcasting.md).

**Trust assumptions:**
- Relay and builders do not leak, copy-trade or front-run the transaction
- Builders the relay reaches win enough slots for timely inclusion

**Threat model:**
- A relay or builder that sees the order can trade ahead of it; contracts and reputation are the only enforcement
- Inclusion waits for a reached builder to win a slot; three builders produced about 88% of MEV-Boost blocks in late September 2026
- Once FOCIL ships, privately routed transactions do not get its inclusion guarantee
- A fallback to the public mempool on relay failure leaks the transaction

**Works best when:**
- The institution already has relay or builder agreements with contractual recourse
- Hiding content from public observers is enough, and builders may see it

**Avoid when:**
- The builder that sees the order may also trade against it
- A protocol-level inclusion guarantee is required

### OTC / Off-chain Matching

```yaml
maturity: production
context: i2i
crops: { cr: medium, o: yes, p: full, s: medium }
uses_patterns: [pattern-network-anonymity]
example_vendors: [renegade, flashbots]
```

**Summary:** Match orders off chain through a private venue or block-builder relationship; settle as a single transaction or bundle that bypasses the public mempool.

**How it works:** Counterparties or solvers match orders in a private venue ([Renegade](../vendors/renegade.md)) and submit the matched trade as a bundle through a [Private Relay](#private-relay), such as [Flashbots](../vendors/flashbots.md) Auction, bypassing the public mempool. Settlement runs on chain; pre-trade content is not seen by mempool observers.

**Trust assumptions:**
- Private venue or builder honesty (no copy-trading from order content)
- Builder relationship for bundle inclusion
- Custody integration with the matching venue

**Threat model:**
- Venue or builder compromise leaks order content
- Builder market concentration creates censor-the-bundle exposure
- Cross-builder routing (failed inclusion) re-exposes the transaction to public mempool

**Works best when:**
- Large institutional trades where spread cost dominates fees
- Counterparties or solver networks already have established custody integration
- Builder market is sufficiently competitive to bound censor risk

**Avoid when:**
- Counterparties are unknown and venue trust cannot be established
- Bundle-inclusion infrastructure is not available on the target chain

### Encrypted Mempools

```yaml
maturity: prototyped
context: both
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-threshold-encrypted-mempool, pattern-pretrade-privacy-encryption]
example_vendors: [shutter]
```

**Summary:** Transactions are encrypted before submission. Decryption happens after ordering is fixed, never before. See [Threshold-Encrypted Mempool](../patterns/pattern-threshold-encrypted-mempool.md) for the underlying primitive and [Pre-trade Privacy Encryption](../patterns/pattern-pretrade-privacy-encryption.md) for its use with RFQ flows.

**How it works:** The user encrypts the transaction to a key held by a threshold committee (in LUCID, a key publisher the user chooses) and submits the ciphertext. Builders and proposers commit to an ordering without seeing content; the key holders then release the key, and the transactions execute in the committed order. [Shutter](../vendors/shutter.md) runs this on Gnosis Chain. On Ethereum no encrypted mempool is live: LUCID ([EIP-8184](https://eips.ethereum.org/EIPS/eip-8184)) is a Draft not scheduled for Hegotá, and out-of-protocol designs depend on builders opting in. On a permissioned chain where one operator sequences, the same construction hides transactions from the operator until the ordering is fixed.

**Trust assumptions:**
- Key holders: a t-of-n committee, or a key publisher the user chooses; a single publisher is one trusted party, as a relay is
- Liveness of the key holders for the reveal step
- Builders and proposers include ciphertexts and respect the encrypt-then-order discipline

**Threat model:**
- Threshold compromise (t collusions) reveals transactions before ordering
- Threshold liveness failure stalls transactions; stalled transactions may eventually leak
- Builders and proposers can still censor by sender, or drop ciphertexts altogether
- Key holders see the committed ordering first and can withhold the key

**Works best when:**
- MEV protection is the primary concern and venue trust is unattractive
- The chain or rollup supports ordering primitives that run over encrypted payloads
- Threshold network trust is administratively manageable
- A permissioned chain's operator sequences transactions and should not see them before ordering

**Avoid when:**
- No threshold network or key publisher is deployed for the target chain (Ethereum L1 as of October 2026)
- Latency budget cannot accommodate the encrypt-decrypt round trip

### Private Rollups

```yaml
maturity: prototyped
context: both
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns: [pattern-privacy-l2s, pattern-shielding]
example_vendors: [aztec, fhenix, zksync, ey]
```

**Summary:** Run trading on a privacy-native rollup (shared) or an enterprise-permissioned privacy rollup (Prividium, Nightfall) where state and mempool are private by construction.

**How it works:** Shared private rollups ([Aztec](../vendors/aztec.md), [Fhenix](../vendors/fhenix.md)) provide hidden state and execution at the protocol layer; institutional desks transact inside the rollup. Enterprise rollups ([ZKsync Prividium](../vendors/zksync.md), [EY Nightfall](../vendors/ey.md#nightfall-v4)) deploy a permissioned privacy rollup with a known operator set. See [Privacy L2s](../patterns/pattern-privacy-l2s.md) for the underlying pattern.

**Trust assumptions:**
- Sequencer trust (centralized in early shared deployments; consortium for enterprise)
- Bridge contract correctness for L1 settlement
- Rollup proving system soundness

**Threat model:**
- Sequencer outage or censorship; escape paths leak linkage during forced exits
- Bridge boundary leaks deposit and withdraw amounts; pair with shielded pools at boundaries
- Enterprise rollup operator set is the adversary in the worst case for permissioned deployments

**Works best when:**
- Comprehensive privacy (mempool + state + execution) is required
- Trading volume justifies committing to a privacy rollup
- Operator trust model fits the institution (consortium or shared)
- End users on shared privacy-native rollups (Aztec, Fhenix) need protocol-default mempool privacy without venue trust; enterprise-permissioned variants (Prividium, Nightfall) remain institutional

**Avoid when:**
- Trading must remain on a public chain
- Rollup decentralization timeline is incompatible with the production timeline

## Comparison

| Axis | Private Relay | OTC / Off-chain Matching | Encrypted Mempools | Private Rollups |
|---|---|---|---|---|
| **Maturity** | production | production | prototyped | prototyped |
| **Context** | both | i2i | both | both |
| **CROPS** | CR:low O:part P:part S:med | CR:med O:y P:full S:med | CR:med O:part P:part S:med | CR:med O:part P:full S:med |
| **Trust model** | Relay and builders | Venue or builder | Key holders (threshold committee) | Sequencer + bridge |
| **Privacy scope** | Pre-inclusion content, hidden from the public only | Pre-trade content only | Pre-inclusion content, hidden from builders too | Mempool + state + execution |
| **Performance** | Waits for a reached builder to win a slot | Minimal latency | Key-release step; next block in LUCID | L2-internal latency |
| **Operator req.** | Relay / builders | Venue / builder | Threshold network | Sequencer |
| **Cost class** | Low | Low | Low | Low (L2-internal) |
| **Regulatory fit** | Strong (unchanged on-chain record) | Strong (audit via venue) | Strong (post-decrypt audit) | Strong (per-rollup view keys) |
| **Failure modes** | Builder front-running; low builder coverage; public fallback | Venue compromise; builder centralization | Threshold compromise; liveness; key withholding | Sequencer outage; bridge exploit; operator capture (enterprise) |

## Persona perspectives

### Business perspective

For institutional desks on public chains, a Private Relay is the production default for orders that must reach the chain: private RPCs and direct builder submission are mature and integrate with custody. OTC / Off-chain Matching suits size-sensitive trades, with Renegade and similar dark-pool venues offering ZK-backed matching. Encrypted Mempools win where builder or venue trust is unattractive and the chain supports threshold-encrypted ordering, which on Ethereum L1 waits for an enshrined design. Private Rollups are the suitable choice when privacy across mempool, state, and execution is required and the institution accepts the rollup operator model. The hybrid is to route by trade size and venue: large blocks through OTC; mid-size through a Private Relay, or an encrypted mempool on chains that run one; flow that needs full privacy through a private rollup.

### Technical perspective

A Private Relay is the lightest engineering integration: point the submission path at a private RPC or builder endpoint. OTC / Off-chain Matching adds a venue integration and ships signed bundles. Encrypted Mempools require a threshold network on the target chain (Shutter on Gnosis Chain; none on Ethereum L1 yet) and tolerating the encrypt-decrypt round trip. Private Rollups are the heaviest commit: deploy or join a privacy-native rollup, integrate with its sequencer model, and accept the bridge boundary as a liquidity surface. None of the four offers cryptographic ordering atomicity across builders or rollups; cross-domain MEV remains an open frontier for chains that have multiple block builders or sequencers competing for inclusion.

### Legal & risk perspective

This is a perspective for legal review by the deploying desk, not legal advice. OTC venues are typically classified under existing dark-pool frameworks; whether a specific venue triggers transaction-reporting or best-execution obligations is for counsel. Encrypted mempools delay disclosure rather than erase it; whether a post-decrypt audit trail satisfies a given reporting framework is a jurisdictional question. Private Rollups raise classification questions about the operator set and the disclosure interface; enterprise rollups (Prividium, Nightfall) are typically deployed in consortium structures, but acceptance varies by regulator. Cross-border deployment surfaces the question of whether a given regulator views encrypted-mempool ordering as compatible with its market-fairness rules; the document does not claim approval in any jurisdiction.

## Recommendation

### Default

For institutional desks on public chains, default to a Private Relay for orders that must reach the chain, and Renegade-class ZK dark-pool matching for size-sensitive trades. Pair with venue-side compliance reporting and custody integration. For chains with mature threshold-encrypted mempools (none on Ethereum L1 as of October 2026), layer Encrypted Mempools as the second-line protection for orders that must reach the public chain.

### Decision factors

- If venue trust is unacceptable and the chain supports threshold-encrypted ordering, choose Encrypted Mempools.
- If comprehensive privacy across mempool, state, and execution is required and the rollup operator model is acceptable, choose Private Rollups.
- If builders may see the order, a Private Relay is enough.
- If trading must remain on a public chain and threshold-encrypted ordering is unavailable, a Private Relay and OTC matching are the options in production; both trust builders or venues with the content.

### Hybrid

Tier the privacy stack by trade size: large blocks (>EUR 5M-equivalent) through OTC; mid-size and time-sensitive flow through a Private Relay, or an encrypted mempool on chains that run one; comprehensive privacy flows (consortium or institutional desk) on a private rollup. Bridge balances out by routing all settlement to a shielded pool boundary so cross-tier movement does not re-expose history.

## Open questions

1. **Regulatory acceptance.** How do regulators view threshold-encrypted ordering and OTC routing relative to traditional mempool transparency?
2. **Market impact.** Effects on price discovery and market structure as private broadcasting share grows.
3. **Cross-chain coordination.** Maintaining privacy when broadcasting across multiple networks; cross-builder MEV is unresolved.
4. **Timing analysis.** Sophisticated adversaries may still extract information from transaction-timing patterns even under encrypted mempools.
5. **Builder market structure.** Concentration at the builder layer creates a censorship and copy-trading risk that economic competition alone may not bound.
6. **Inclusion or privacy.** Until inclusion lists can carry sealed transactions, a private route gives up FOCIL's inclusion guarantee. How should a sender weigh that for compliance-critical transactions such as freezes?
7. **Key publishers.** Who may hold decryption keys for regulated flow, and how is a key holder that withholds after seeing the ordering penalised?
8. **Permissioned operators.** On a permissioned chain the operator that sequences sees every pending transaction. Encrypted submission hides it until ordering, not after. Is that enough without application-layer confidentiality?

