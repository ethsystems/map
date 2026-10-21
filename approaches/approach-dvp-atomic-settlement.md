---
title: "Approach: Atomic DvP Settlement"
status: ready
last_reviewed: 2026-10-21

use_case: private-bonds
related_use_cases: [private-derivatives, private-corporate-bonds, private-government-debt, private-rwa-tokenization, private-repo]

primary_patterns:
  - pattern-dvp-erc7573
supporting_patterns:
  - pattern-commit-and-prove
  - pattern-erc3643-rwa
  - pattern-mpc-custody
  - pattern-private-pvp-stablecoins-erc7573
  - pattern-tee-zk-settlement
  - pattern-shielding

pocs:
  folder: pocs/private-trade-settlement
  requirements: pocs/private-trade-settlement/REQUIREMENTS.md
  pocs:
    - name: "TEE+ZK Cross-Chain Swap"
      sub_approach: "Conditional Transfer with Oracle"
      spec: pocs/private-trade-settlement/tee_swap/SPEC.md
      status: implemented

open_source_implementations:
  - url: https://github.com/chatch/hashed-timelock-contract-ethereum
    description: "Hashed Timelock Contracts for ETH, ERC-20, and ERC-721 on Ethereum"
    language: Solidity
---

# Approach: Atomic DvP Settlement

## Problem framing

### Scenario

An asset manager sells a EUR 1M corporate bond position to a bank on an L2 network. Both parties want assurance that the bond and the EURC payment exchange atomically: either both legs settle or neither does, with deterministic recovery if a leg fails. The same atomic-settlement primitive generalizes to other instrument types.

### Requirements

- Both legs complete atomically or neither does; no partial settlement
- Counterparty risk is structurally eliminated (assets in escrow, not held by the other party)
- Deterministic conditions and timeouts; clear failure semantics for legal review
- Optional: compatible with ERC-6123 (derivative lifecycle, Draft) and ERC-3643 (securities token, Final)

### Constraints

- Single-network DvP is atomic by construction: one transaction settles both legs or reverts. Cross-network atomicity is the hard case and needs a trusted intermediary, an oracle, or a hashlock + timeout (HTLC)
- The cross-network and HTLC paths lock capital for the settlement or timeout window; lockup duration is a design parameter specific to those paths
- Escrow contracts are practically immutable in production; upgrades imply migration

## Approaches

### Hash Time-Locked Contracts (HTLC)

```yaml
maturity: prototyped
context: i2i
crops: { cr: high, o: yes, p: none, s: high }
uses_patterns: [pattern-dvp-erc7573]
example_vendors: []
```

**Summary:** Payment and asset are each locked to a hash; revealing the preimage releases both legs. Primarily a cross-chain primitive; for single-network DvP a native atomic swap or escrow is simpler and leaks less.

**How it works:** The seller generates secret S and shares H(S). The buyer locks payment with H(S) and timeout T1; the seller locks the asset with H(S) and timeout T2 > T1. The seller reveals S to claim payment before T1, which makes S public; the buyer then uses S to claim the asset before the later timeout T2. The secret-revealer's own leg must expire last, so the counterparty always has time to claim once S is public. If S is never revealed, both sides refund at their respective timeouts.

**Trust assumptions:**
- Hash function preimage resistance
- L1 (or L2) consensus for transaction inclusion within timeout windows
- No trusted third party

**Threat model:**
- Network congestion at the timeout boundary can strand a leg
- T2 > T1 ordering must hold (the secret-revealer's leg expires last); mis-parameterization breaks atomicity
- Free-option problem: the holder of S chooses whether to reveal, which has implicit option value

**Works best when:**
- The two legs sit on different chains and each chain supports hashlock + timeout (HTLC's distinctive niche is cross-chain)
- Counterparties are untrusted and no shared escrow contract is available
- Public visibility of the hashlock, preimage, amounts, and parties is acceptable

**Avoid when:**
- Both legs are on one network: a single atomic swap or escrow transaction settles atomically with no lockup and less leakage (add a shielded pool for privacy)
- Trade details must stay private: HTLC publishes the hashlock, preimage, amounts, and counterparties
- Free-option exposure must be eliminated (use escrow instead)
- Assets do not tolerate the lockup window required by safe timeout margins

### Escrow with Dual Approval

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: yes, p: partial, s: high }
uses_patterns: [pattern-mpc-custody, pattern-erc3643-rwa]
example_vendors: []
```

**Summary:** Both legs are deposited to an escrow contract; release requires approvals from both parties or, on dispute, an arbitrator decision.

**How it works:** The escrow holds both legs atomically. After deposits and off-chain or oracle-attested condition verification, both parties sign a release transaction; the escrow atomically swaps the legs. A configurable dispute window allows arbitration before final release.

**Trust assumptions:**
- Escrow contract correctness (audited, ideally formally verified)
- Arbitrator (if invoked) is trusted by both parties
- Oracle correctness if external conditions gate release

**Threat model:**
- Counterparty refuses to sign release; recovery via arbitrator or timeout
- Arbitrator collusion or compromise distorts disputed releases
- Escrow bug results in asset loss; mitigated by audits and insurance

**Works best when:**
- Known counterparties with established legal relationships
- Settlement involves human-in-the-loop conditions or off-chain verification
- Dispute-resolution paths must be explicit for legal review

**Avoid when:**
- Trustless settlement is required and arbitrator dependency is unacceptable
- Settlement must be sub-second; arbitration adds latency

### Conditional Transfer with Oracle

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-dvp-erc7573, pattern-private-pvp-stablecoins-erc7573, pattern-tee-zk-settlement, pattern-shielding]
poc_spec: pocs/private-trade-settlement/tee_swap/SPEC.md
example_vendors: []
```

**Summary:** Asset and payment are locked; an oracle attests to an external condition; attestation triggers atomic settlement.

**How it works:** Both legs are deposited into escrow with a condition predicate (price bound, regulatory event, payment confirmation). An oracle monitors the condition and submits an attestation when it is met; the escrow validates the attestation and executes the atomic swap. A fallback path handles oracle non-response.

**Trust assumptions:**
- Oracle integrity (single oracle or quorum, depending on design)
- Condition specification is unambiguous and enforceable
- Fallback path is funded and reachable

**Threat model:**
- Oracle compromise or compelled disclosure misroutes settlement
- Condition ambiguity drives disputes; mitigated by precise specification and arbitration
- Oracle outage blocks settlement; mitigated by multi-oracle quorum and manual fallback

**Works best when:**
- Settlement is event-driven (derivatives expiry, cross-network payment confirmation)
- Cross-network DvP is needed and ERC-7573 oracle is acceptable
- Condition can be expressed as a verifiable attestation

**Avoid when:**
- Trusted oracle is incompatible with the threat model
- Conditions cannot be specified precisely enough for unambiguous attestation

**Implementation notes:** PoC at [TEE+ZK Cross-Chain Swap](https://github.com/ethsystems/pocs/tree/master/pocs/private-trade-settlement/tee_swap) implements the oracle-attested variant where a TEE coordinator stands in for the oracle, generates a zero-knowledge proof of correct execution, and reveals stealth keys on success. See [Approach: Private Trade Settlement, TEE+ZK Coordination](approach-private-trade-settlement.md) for the same primitive in cross-network context. Common escrow-condition shapes:

| Class | Example |
|---|---|
| Time-based | Settlement date (T+2 bond settlement); lock period (vesting); expiry (failed-trade cleanup) |
| Event-based | Payment finality confirmation; regulatory approval; multi-sig threshold approval |
| Value-based | Price-bound limit orders; collateral-ratio margin calls; net settlement against batched obligations |

## Comparison

| Axis | HTLC | Escrow with Dual Approval | Conditional Transfer with Oracle |
|---|---|---|---|
| **Maturity** | prototyped | prototyped | prototyped |
| **Context** | i2i | i2i | i2i |
| **CROPS** | CR:hi O:y P:none S:hi | CR:med O:y P:part S:hi | CR:med O:part P:part S:med |
| **Trust model** | Trustless (hash + timeout) | Counterparties + arbitrator | Oracle |
| **Privacy scope** | None on chain: hashlock, preimage, amounts, parties visible | Public escrow; party identities visible | Public escrow + oracle attestation |
| **Performance** | Two transactions per leg | Deposit + dual signature + release | Deposit + attestation + release |
| **Operator req.** | None | Arbitrator on dispute | Oracle (single or quorum) |
| **Cost class** | Low | Medium (dispute path) | Medium (oracle infra) |
| **Regulatory fit** | Strong for trustless contexts | Strong with named arbitrator | Conditional on oracle governance |
| **Failure modes** | Network congestion at timeout; free-option | Counterparty stalls; arbitrator compromise | Oracle outage or compromise |

## Persona perspectives

### Business perspective

On a single network, DvP is atomic by construction: one transaction settles both legs or reverts. The remaining design work is the release logic: the conditions and compliance checks that gate settlement.

For institutional settlement among known counterparties, Escrow with Dual Approval is the default: it matches the human-in-the-loop nature of traditional settlement, accommodates compliance and operational checks before release, and provides an explicit dispute path that legal review can rely on. Conditional Transfer with Oracle suits event-driven flows (regulatory approval, payment-finality confirmation, cross-network triggers) where the release condition is external; the operational burden is the oracle governance rather than the contract itself. HTLC is a cross-chain primitive, relevant only when the two legs sit on different chains; institutional desks rarely use it for single-network DvP, where a native swap or escrow is simpler and the hashlock would otherwise publish the trade.

On any of these paths, privacy comes from settling the legs inside a shielded pool; the atomicity mechanism alone leaves the trade visible.

### Technical perspective

Single-network atomicity is free: a single contract call that swaps both legs either commits or reverts, with no capital lockup. The engineering question is the release logic.

Escrow is the most legible: deposit, verify, release, refund, with a clear state machine per branch. Conditional Transfer with Oracle adds the oracle-integration surface (push vs pull, single vs quorum, fallback semantics) and inherits the attestation infrastructure's failure modes. HTLC is the cross-chain option and the trickiest to parameterize: T2 > T1 with margins for network congestion is load-bearing, a stalled leg has no recovery beyond timeout, and the preimage publishes the trade, so it is reserved for the genuinely cross-chain case.

Cross-network atomicity is the open frontier: ERC-7573 is the working draft, but trustless cross-chain DvP remains unresolved (see [Private Trade Settlement](approach-private-trade-settlement.md) for the privacy rails).

### Legal & risk perspective

This is a perspective for legal review by the deploying institution, not legal advice.

Escrow with a named arbitrator references existing arbitration frameworks (LCIA, ICC); whether the chosen framework binds in cross-border settlement is a question for counsel. Conditional Transfer with Oracle raises a classification question about the oracle's role (data provider, attestation issuer, fiduciary?) and the audit access to its evidence trail. HTLC, where used for a cross-chain leg, has a deterministic outcome (preimage revealed or timeout) that can be documented precisely, but it publishes the trade on chain; whether that documentation and disclosure suit a specific dispute regime is for legal review.

For each option, the dispute and recovery path (arbitrator decision, oracle non-response, timeout refund, escrow bug) would need to be modelled explicitly under the applicable law.

## Recommendation

### Default

For institutional bond DvP among known counterparties on a single network, default to Escrow with Dual Approval, integrated with ERC-3643 for security tokens. The arbitration path is named in the trade documentation; the oracle (if invoked) is scoped to specific external triggers (e.g., regulatory approval, payment finality).

### Decision factors

- If settlement is event-driven and the trigger is external (regulatory approval, payment finality, cross-network confirmation), choose Conditional Transfer with Oracle, ideally via ERC-7573.
- If the two legs sit on different chains, use HTLC or oracle/intermediary coordination and accept that the hashlock publishes the trade; trustless cross-chain DvP remains an open problem (see [Private Trade Settlement](approach-private-trade-settlement.md)).
- If amounts and counterparties must stay private, settle the legs inside a shielded pool rather than relying on the atomicity mechanism for privacy.

### Hybrid

Bond DvP can run primary settlement through Escrow with Dual Approval and fall back to a timed unilateral refund path on counterparty non-response, with an oracle attesting to payment finality on an external rail. For privacy, the escrow legs settle inside a shielded pool so trade IDs and party identities are not exposed on chain.

## Open questions

1. **Trustless cross-network DvP.** Without a trusted oracle or intermediary, cross-chain atomicity is unsolved; ERC-7573 acknowledges this as an open problem.
2. **Free-option pricing.** HTLC's free-option problem is well-known but underpriced in institutional contexts; how should it be hedged or contractually offset?
3. **Oracle governance for institutional DvP.** What governance model satisfies regulator audit expectations for ERC-7573 oracles?
4. **Privacy of escrow state.** Standard escrow contracts publish trade IDs and party identities; privacy-preserving DvP requires composing with shielded pools or [Commit and Prove](../patterns/pattern-commit-and-prove.md).
5. **Settlement-window calibration.** Network congestion margins for HTLC timeouts and escrow dispute windows are deployment-specific; no canonical guidance.

