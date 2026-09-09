---
title: "Approach: Private Trade Settlement"
status: ready
last_reviewed: 2026-08-11

use_case: private-bonds
related_use_cases: [private-derivatives, private-rwa-tokenization, private-corporate-bonds, private-government-debt, private-stocks, private-commodities, private-fx, private-repo]

primary_patterns:
  - pattern-shielding
  - pattern-privacy-l2s
  - pattern-tee-zk-settlement
  - pattern-cross-chain-privacy-bridge
supporting_patterns:
  - pattern-dvp-erc7573
  - pattern-regulatory-disclosure-keys-proofs
  - pattern-private-pvp-stablecoins-erc7573

pocs:
  folder: pocs/private-trade-settlement
  requirements: pocs/private-trade-settlement/REQUIREMENTS.md
  pocs:
    - name: "TEE+ZK Cross-Chain Swap"
      sub_approach: "TEE+ZK Coordination"
      spec: pocs/private-trade-settlement/tee_swap/SPEC.md
      status: implemented

open_source_implementations:
  - url: https://github.com/Railgun-Privacy/contract
    description: "Railgun shielded pool with Adapt Modules for DeFi composability"
    language: Solidity
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec privacy-native L2"
    language: TypeScript / Noir
  - url: https://github.com/across-protocol/contracts
    description: "Across (ERC-7683 intents-based cross-chain settlement)"
    language: Solidity
---

# Approach: Private Trade Settlement

## Problem framing

### Scenario

A treasury desk and a counterparty bank settle a EUR 25M corporate bond against EURC. Both parties want hidden amounts and counterparty identities, atomic execution, and selective regulator access. Some trades involve a bond on a privacy L2 against cash on Ethereum L1, forcing cross-network coordination.

### Requirements

- Atomic DvP: both legs complete or neither does (atomicity primitives covered in [Atomic DvP Settlement](approach-dvp-atomic-settlement.md))
- Hide amounts, counterparties, and trade size
- Selective disclosure for regulators (per-jurisdiction, time-bound)
- Composition with custody and KYC workflows

### Constraints

- Single-network atomicity is inherent in transaction execution; cross-network atomicity requires trust assumptions
- Privacy set size on a permissioned pool is bounded by the institutional participant count
- Bridge boundaries leak deposit/withdraw amounts unless paired with shielded pools on each side

## Approaches

### UTXO Shielded Pool DvP

```yaml
maturity: production
context: i2i
crops: { cr: high, o: yes, p: full, s: high }
uses_patterns: [pattern-shielding, pattern-regulatory-disclosure-keys-proofs]
example_vendors: [railgun, paladin, privacypools]
```

**Summary:** Both counterparties hold assets in a shielded pool; settlement runs as a coordinated JoinSplit that consumes input notes and produces output notes in a single atomic transaction.

**How it works:** Each party deposits to the shielded pool; trade execution is a multi-input, multi-output JoinSplit proven in zero knowledge: bond notes and cash notes are consumed, new owner-rotated notes are created. The verifier contract checks Merkle membership, nullifier uniqueness, and conservation of value across asset classes. Per-note viewing keys enable selective disclosure to regulators.

**Trust assumptions:**
- L1 consensus and verifier contract correctness
- Gas relayer for liveness on private withdrawals
- Per-pool issuer (if KYC-gated entry is enforced)

**Threat model:**
- Linkability at deposit / unshield boundaries to public ERC-20
- Anonymity-set economics: small institutional pools weaken anonymity even if cryptography is sound
- Relayer censorship; mitigated by direct withdraw at the cost of address linkage

**Works best when:**
- Both legs (asset and cash) are tokenized and can coexist on the same network
- Institution accepts the gas envelope of L1 verification
- Production maturity matters and Railgun-class infrastructure is acceptable

**Avoid when:**
- One leg requires a network where the shielded pool is not deployed
- Anonymity-set on the available pool is too small to mask trade size

### Privacy L2 Native DvP

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns: [pattern-privacy-l2s, pattern-regulatory-disclosure-keys-proofs]
example_vendors: [aztec, miden]
```

**Summary:** Asset and payment contracts run on a privacy-native rollup; DvP is a private contract call against encrypted state, no shielding overhead.

**How it works:** Bond notes and cash notes are native private notes in the rollup's execution model. A private DvP contract call consumes both and produces new notes for each party. Validity proofs settle on L1; Incoming Viewing Keys (IVKs) provide account-level disclosure.

**Trust assumptions:**
- Sequencer for ordering (currently centralized in early deployments)
- Bridge contracts for L1 settlement
- Rollup proving system soundness

**Threat model:**
- Sequencer outage forces escape-game exits with weakened privacy
- Bridge boundary exposes deposit/withdraw amounts
- IVK compromise reveals account-level flows

**Works best when:**
- Both legs can be deployed natively on the privacy L2
- Bond logic benefits from native private primitives (lifecycle, coupons)
- Rollup decentralization roadmap is acceptable

**Avoid when:**
- Cross-network legs are unavoidable
- Production timeline cannot wait for the rollup's decentralization milestones

### TEE+ZK Coordination

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns: [pattern-tee-zk-settlement, pattern-cross-chain-privacy-bridge]
poc_spec: pocs/private-trade-settlement/tee_swap/SPEC.md
example_vendors: []
```

**Summary:** A TEE coordinator matches orders privately, generates zero-knowledge proofs of correct execution, and coordinates cross-chain settlement using stealth addresses with timeout-based refunds.

**How it works:** Counterparties submit encrypted orders to an attested TEE. The TEE matches, validates funds (via cross-chain proofs), and locks notes to stealth addresses on each chain with dual conditions (TEE-revealed key OR timeout refund). On success, the TEE generates a SNARK of correct execution and reveals stealth keys; on failure, both sides reclaim after timeout. EIP-5564 stealth addresses anchor the destination side.

**Trust assumptions:**
- TEE vendor and remote-attestation chain
- Stealth-address key generation soundness
- Bridge contracts on each chain

**Threat model:**
- Hardware vendor can observe plaintext during execution
- TEE crash mid-settlement is recoverable via timeout but not atomic
- Side-channel attacks on the TEE class

**Works best when:**
- Hardware trust is already accepted (HSM infrastructure, custodial environments)
- Cross-network settlement is unavoidable
- Timeout-based recovery is acceptable for the trade type

**Avoid when:**
- Threat model excludes hardware-vendor trust
- True cryptographic atomicity across chains is required

### MPC/Threshold Settlement

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-cross-chain-privacy-bridge]
example_vendors: []
```

**Summary:** A distributed MPC committee coordinates cross-chain settlement; threshold signatures authorize transfers on each chain with slashing for misbehavior.

**How it works:** Counterparties send encrypted orders to the committee. The committee verifies funds, runs the matching logic under MPC, and produces threshold signatures that authorize the asset and payment legs on their respective chains. Misbehavior is detected via attestations and slashes staked collateral.

**Trust assumptions:**
- Honest majority among MPC nodes (t-of-n)
- Staked collateral is sufficient relative to trade values
- Liveness of the committee for settlement windows

**Threat model:**
- Collusion above threshold breaks confidentiality and atomicity
- Slashing is economic, not cryptographic; large-value trades may exceed collateral
- Committee liveness boundary is an availability risk

**Works best when:**
- Decentralized cross-chain coordination without hardware trust is required
- Auditability of the coordination layer is mandated by the disclosure model
- Trade sizes are bounded by available staked collateral

**Avoid when:**
- Honest-majority assumption among MPC nodes is unacceptable
- Trade sizes exceed feasible collateral pools

### Intent-Based Settlement (ERC-7683)

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: yes, p: partial, s: medium }
uses_patterns: [pattern-cross-chain-privacy-bridge]
example_vendors: []
```

**Summary:** Counterparties express settlement intents; competitive solvers fill orders using their own capital across chains, bearing execution risk.

**How it works:** A user publishes an ERC-7683 intent (asset, amount, acceptable rate). Solvers compete to fill the destination leg from their own inventory; the protocol reimburses the winning solver from the source leg after fill. Solver collateral and reputation gate participation. Settlement on the destination chain is fast; source-side reimbursement runs over a slower verification path.

**Trust assumptions:**
- Solver collateral is adequate for the trade size
- Reputation systems and slashing capture misbehavior
- Source-side proof of fill is verifiable

**Threat model:**
- Intents are visible to solvers; privacy-preserving solver discovery is an open problem
- Solver griefing or last-look behavior; mitigated by competition and slashing
- Source-side proof failure can lock funds in dispute

**Works best when:**
- Asynchronous settlement with slight timing flexibility is acceptable
- Competitive execution and price improvement matter
- Counterparties are content to expose order details to solvers

**Avoid when:**
- Trade order or counterparty must remain private from the solver network
- Strict atomicity is required; intent-based settlement is economic, not cryptographic

## Comparison

| Axis | UTXO Shielded Pool | Privacy L2 | TEE+ZK | MPC/Threshold | Intent-Based |
|---|---|---|---|---|---|
| **Maturity** | production | documented | prototyped | documented | prototyped |
| **Context** | i2i | i2i | i2i | i2i | i2i |
| **CROPS** | CR:hi O:y P:full S:hi | CR:med O:part P:full S:med | CR:med O:part P:full S:med | CR:med O:part P:part S:med | CR:med O:y P:part S:med |
| **Trust model** | Cryptographic only | Sequencer + bridge | Hardware vendor | Honest-majority MPC | Solver collateral + reputation |
| **Privacy scope** | Amounts + counterparties + addresses | Full state on L2 | Encrypted coordination, stealth destinations | Strong (MPC) | Intents visible to solvers |
| **Performance** | L1 gas; verification cost | L2-internal fees | Enclave speed | MPC rounds | Variable (solver competition) |
| **Operator req.** | None (gas relayer optional) | Yes (sequencer) | Yes (TEE coordinator) | Yes (MPC committee) | Yes (solver network) |
| **Cost class** | Medium-high | Low | Low (per trade), variable infra | Medium | Variable, market-driven |
| **Regulatory fit** | Strong (per-note view keys) | Strong (IVKs) | Conditional (vendor attestation) | Strong (MPC audit) | Weak unless layered with private rails |
| **Failure modes** | Anonymity-set too small; relayer censor | Sequencer outage; bridge exploit | TEE crash; side-channel | Collusion above threshold | Solver griefing; intent leakage |

## Persona perspectives

### Business perspective

For institutional same-network settlement among known counterparties, UTXO Shielded Pool DvP is the default: production maturity, vendor coverage, and a disclosure interface that maps onto eWpG / MiCA. Privacy L2 Native DvP fits where bond logic is complex and the rollup's decentralization timeline is acceptable. Cross-network settlement carries unavoidable trust: TEE+ZK suits institutions with hardware trust already in scope; MPC/Threshold fits decentralized coordination without hardware vendors; Intent-Based is the suitable choice for asynchronous flows where competitive execution matters more than order privacy. The default is to keep both legs on one network whenever possible; only resort to cross-network when liquidity or regulation forces it.

### Technical perspective

Same-network settlement is the simpler engineering surface: deploy a verifier and a relayer (UTXO) or rely on rollup primitives (Privacy L2). Cross-network settlement multiplies the surface, TEE attestation chains, MPC committee operations, or solver network integration each carry their own reliability and audit burden. None of the cross-network options offers true atomic settlement; each replaces cryptographic atomicity with a different recovery path: timeout (TEE), slashing (MPC), or solver risk (intent). Realtime proving and based sequencing are the research frontiers that may eventually deliver trustless cross-rollup atomicity but are not production-ready.

### Legal & risk perspective

This is a perspective for legal review by the deploying institution, not legal advice. Same-network UTXO and Privacy L2 settlement expose viewing keys as the disclosure interface and a defined audit fingerprint (nullifiers, sequencer events); whether that interface satisfies the applicable settlement-reporting framework is a question for jurisdictional review. Cross-network options surface additional questions: TEE attestation chains and vendor governance, MPC committee membership and slashing logs, solver-network operations and intent transparency. Each raises classification questions (who is the legal entity in the coordination layer, what evidence does an auditor need, what is the dispute path on failure) that counsel would resolve per jurisdiction. The recovery semantics differ materially across cross-network options (timeout refund for TEE, slashing payout for MPC, solver loss for intent-based) and would need to be modelled explicitly under the applicable law before sign-off.

## Recommendation

### Default

For institutional trade settlement where both legs can coexist on a single network, default to UTXO Shielded Pool DvP on a production shielded pool ([Railgun](../vendors/railgun.md), [Paladin](../vendors/paladin.md), or [Privacy Pools](../vendors/privacypools.md)). Settlement runs as a coordinated JoinSplit; selective disclosure runs through per-note viewing keys; the dispute path is the standard institutional arbitration framework.

### Decision factors

- If both legs cannot coexist on a single network and hardware trust is acceptable, choose TEE+ZK Coordination.
- If decentralized cross-chain coordination without hardware vendors is required, choose MPC/Threshold Settlement and validate that staked collateral covers the trade-size envelope.
- If the flow is asynchronous and competitive execution matters more than order privacy, choose Intent-Based Settlement (ERC-7683).
- If bond logic is complex and the rollup's decentralization timeline is acceptable, choose Privacy L2 Native DvP.

### Hybrid

Run primary settlement on a shielded pool for liquidity-dense flow; pair with TEE+ZK for occasional cross-network legs where one asset class is unavailable on the primary network. Selective disclosure runs uniformly through the regulator-disclosure-keys pattern across both rails. Where both shielded pool and privacy L2 are available, route trade size to the venue that satisfies the anonymity-set bound for that trade.

## Open questions

1. **Privacy-set economics.** Minimum shielded pool size for adequate anonymity at institutional trade sizes is unmeasured.
2. **Cross-chain necessity.** Can tokenized deposit networks (EURC, USDC) deploy on privacy L2s, eliminating cross-chain cash legs?
3. **Regulatory acceptance per trust model.** Which trust models (TEE, MPC, economic) satisfy regulatory requirements across jurisdictions?
4. **Solver privacy.** Can intent-based architectures preserve order privacy while enabling competitive execution?
5. **Realtime proving / based sequencing.** Whether synchronous cross-rollup composability can deliver trustless cross-network atomicity within block time.

