---
title: "Approach: Private Transaction Broadcasting"
status: ready
last_reviewed: 2026-06-24

use_case: private-stocks
related_use_cases: [private-derivatives, private-fx, private-bonds, private-stablecoins]

primary_patterns:
  - pattern-pretrade-privacy-encryption
  - pattern-privacy-l2s
supporting_patterns:
  - pattern-shielding
  - pattern-network-anonymity

open_source_implementations:
  - url: https://github.com/flashbots/suave-geth
    description: "SUAVE private-intent execution (suave-geth archived May 2025; Flashbots migrated to BuilderNet)"
    language: Go
  - url: https://github.com/shutter-network/shutter
    description: "Shutter Network (threshold-encrypted mempool)"
    language: Rust / Go
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec privacy-native L2"
    language: TypeScript / Noir
---

# Approach: Private Transaction Broadcasting

## Problem framing

### Scenario

A trading desk routes large institutional orders against on-chain venues. Visible transactions in the public mempool enable front-running, sandwiching, and intent inference; the spread cost on a EUR 25M order is large enough to dominate fees for the year. The desk needs broadcasting infrastructure that hides the order content, the timing, or both, while preserving validity and ordering guarantees.

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

## Approaches

### OTC / Off-chain Matching

```yaml
maturity: production
context: i2i
crops: { cr: medium, o: yes, p: full, s: medium }
uses_patterns: [pattern-network-anonymity]
example_vendors: [renegade, flashbots]
```

**Summary:** Match orders off chain through a private venue or block-builder relationship; settle as a single transaction or bundle that bypasses the public mempool.

**How it works:** Counterparties or solvers match orders in a private venue ([Renegade](../vendors/renegade.md), [Flashbots](../vendors/flashbots.md)) and submit the matched trade directly to a block builder, bypassing the public mempool. Settlement runs on chain; pre-trade content is not seen by mempool observers.

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
crops: { cr: high, o: yes, p: full, s: high }
uses_patterns: [pattern-pretrade-privacy-encryption]
example_vendors: [shutter]
```

**Summary:** Transactions are encrypted before submission. Decryption happens after ordering is fixed, never before. See [Pre-trade Privacy Encryption](../patterns/pattern-pretrade-privacy-encryption.md) for the underlying primitive.

**How it works:** The user encrypts the transaction or expresses an intent under a private execution environment ([Shutter](../vendors/shutter.md); the earlier SUAVE design has been superseded by Flashbots BuilderNet). The proposer orders submissions before any decryption. A threshold committee decrypts afterwards network. Content is not visible until the ordering is committed.

**Trust assumptions:**
- Threshold key holders (t-of-n) for decryption
- Liveness of the threshold network for the reveal step
- Proposer respects the encrypt-then-order discipline

**Threat model:**
- Threshold compromise (t collusions) reveals transactions before ordering
- Threshold liveness failure stalls transactions; stalled transactions may eventually leak
- Proposer can still censor-by-omission even without seeing content

**Works best when:**
- MEV protection is the primary concern and venue trust is unattractive
- The chain or rollup supports ordering primitives that run over encrypted payloads
- Threshold network trust is administratively manageable

**Avoid when:**
- Threshold network is not deployed for the target chain
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

| Axis | OTC / Off-chain Matching | Encrypted Mempools | Private Rollups |
|---|---|---|---|
| **Maturity** | production | prototyped | prototyped |
| **Context** | i2i | both | both |
| **CROPS** | CR:med O:y P:full S:med | CR:hi O:y P:full S:hi | CR:med O:part P:full S:med |
| **Trust model** | Venue or builder | Threshold key holders | Sequencer + bridge |
| **Privacy scope** | Pre-trade content only | Pre-inclusion content | Mempool + state + execution |
| **Performance** | Minimal latency | Encrypt-decrypt round trip | L2-internal latency |
| **Operator req.** | Venue / builder | Threshold network | Sequencer |
| **Cost class** | Low | Low | Low (L2-internal) |
| **Regulatory fit** | Strong (audit via venue) | Strong (post-decrypt audit) | Strong (per-rollup view keys) |
| **Failure modes** | Venue compromise; builder centralization | Threshold compromise; liveness | Sequencer outage; bridge exploit; operator capture (enterprise) |

## Persona perspectives

### Business perspective

For institutional desks executing large trades on public chains, OTC / Off-chain Matching is the default for production deployments: Flashbots-class infrastructure is mature, Renegade and similar dark-pool venues offer ZK-backed matching, and the operational integration with custody is well-trodden. Encrypted Mempools win where venue trust is unattractive and the chain supports threshold-encrypted ordering. Private Rollups are the suitable choice when comprehensive privacy across mempool, state, and execution is required and the institution accepts the rollup operator model. The hybrid is to route by trade size and venue: large blocks through OTC, mid-size through encrypted mempools, comprehensive flow through a private rollup.

### Technical perspective

OTC / Off-chain Matching is the lightest engineering integration: connect to a venue or builder relay, ship signed bundles. Encrypted Mempools require integrating with the threshold network (Shutter SDK) and tolerating the encrypt-decrypt round trip. Private Rollups are the heaviest commit: deploy or join a privacy-native rollup, integrate with its sequencer model, and accept the bridge boundary as a liquidity surface. None of the three offers cryptographic ordering atomicity across builders or rollups; cross-domain MEV remains an open frontier for chains that have multiple block builders or sequencers competing for inclusion.

### Legal & risk perspective

This is a perspective for legal review by the deploying desk, not legal advice. OTC venues are typically classified under existing dark-pool frameworks; whether a specific venue triggers transaction-reporting or best-execution obligations is for counsel. Encrypted mempools delay disclosure rather than erase it; whether a post-decrypt audit trail satisfies a given reporting framework is a jurisdictional question. Private Rollups raise classification questions about the operator set and the disclosure interface; enterprise rollups (Prividium, Nightfall) are typically deployed in consortium structures, but acceptance varies by regulator. Cross-border deployment surfaces the question of whether a given regulator views encrypted-mempool ordering as compatible with its market-fairness rules; the document does not claim approval in any jurisdiction.

## Recommendation

### Default

For institutional desks on public chains, default to OTC / Off-chain Matching through Flashbots-class infrastructure for large blocks and Renegade-class ZK dark-pool matching for size-sensitive trades. Pair with venue-side compliance reporting and custody integration. For chains with mature threshold-encrypted mempools, layer Encrypted Mempools as the second-line protection for orders that must reach the public chain.

### Decision factors

- If venue trust is unacceptable and the chain supports threshold-encrypted ordering, choose Encrypted Mempools.
- If comprehensive privacy across mempool, state, and execution is required and the rollup operator model is acceptable, choose Private Rollups.
- If trading must remain on a public chain and threshold-encrypted ordering is unavailable, OTC is the only option that has reached production at this layer.

### Hybrid

Tier the privacy stack by trade size: large blocks (>EUR 5M-equivalent) through OTC; mid-size and time-sensitive flow through an encrypted mempool; comprehensive privacy flows (consortium or institutional desk) on a private rollup. Bridge balances out by routing all settlement to a shielded pool boundary so cross-tier movement does not re-expose history.

## Open questions

1. **Regulatory acceptance.** How do regulators view threshold-encrypted ordering and OTC routing relative to traditional mempool transparency?
2. **Market impact.** Effects on price discovery and market structure as private broadcasting share grows.
3. **Cross-chain coordination.** Maintaining privacy when broadcasting across multiple networks; cross-builder MEV is unresolved.
4. **Timing analysis.** Sophisticated adversaries may still extract information from transaction-timing patterns even under encrypted mempools.
5. **Builder market structure.** Concentration at the builder layer creates a censorship and copy-trading risk that economic competition alone may not bound.

