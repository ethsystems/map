---
title: "Pattern: Privacy L2s"
status: ready
maturity: production
type: standard
layer: L2
last_reviewed: 2026-10-21

works-best-when:
  - Strong privacy is required beyond value-hiding, including identity unlinkability and programmable access controls.
  - Institutions want to embed compliance hooks (view keys, audit proofs) directly into the execution environment.

avoid-when:
  - A conservative, battle-tested L1 deployment path is required today.
  - Only minimal value privacy is needed and bridging overhead is unjustified.

context: both
context_differentiation:
  i2i: "Institutions can absorb bridging friction and centralized-sequencer risk as known operational costs. The primary value is programmable private state: confidential contract logic, private balances with DvP hooks, and selective disclosure integrated at the protocol level. Both counterparties share a legal framework, so temporary sequencer centralization is a liveness concern, not an existential one."
  i2u: "End users face asymmetric exposure to the sequencer. A centralized sequencer can unilaterally exclude user transactions; without forced withdrawal via the L1 bridge, user funds can be stranded. Permissionless sequencer selection and forced-exit guarantees are prerequisites before this pattern is safe for user-facing deployments."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Most privacy rollups today use centralized sequencing with no public censorship proofs. Reaches `high` when sequencer selection becomes permissionless leader election with enforceable L1 fallback. Drops to `low` on operator-controlled deployments with no forced-exit path."
  o: "Some privacy L2 stacks are open source; others rely on proprietary provers or closed coprocessors. Reaches `yes` when the full node, prover, and circuit code are published under permissive or copyleft licenses with binding commitments against proprietary prover dependencies."
  p: "Native privacy for amounts, sender, and receiver within the L2. Metadata (bridge activity, L1 anchoring timing) is visible on L1 and must be covered separately if metadata privacy matters."
  s: "Rides on L1 security for the bridge, on the rollup's proof or FHE correctness, and on sequencer availability. Reaches `high` with committee-based state attestation (randomly selected validators signing each checkpoint) and decentralized sequencing."

post_quantum:
  risk: high
  vector: "EC-based proof systems (Groth16, PLONK/KZG) dominate privacy rollup stacks and are broken by CRQC. FHE-based L2s depend on lattice hardness and are PQ-leaning, but the surrounding bridge and signature schemes are typically classical."
  mitigation: "STARK-based rollups with hash commitments for PQ-secure proofs; hybrid PQ-secure bridges for cross-layer messages."

visibility:
  counterparty: [amounts, identities]
  chain: [state_root_anchors]
  regulator: [full_state with viewing key or disclosure proof]
  public: [L1_anchor_commitments]

standards: [ERC-20, ERC-3643, ERC-7573]

related_patterns:
  composes_with: [pattern-shielding, pattern-regulatory-disclosure-keys-proofs, pattern-dvp-erc7573, pattern-erc3643-rwa]
  alternative_to: [pattern-plasma-stateless-privacy]
  see_also: [pattern-forced-withdrawal, pattern-user-controlled-viewing-keys]

open_source_implementations:
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec Network: privacy L2 with native shielding"
    language: "Noir, Rust, TypeScript"
  - url: https://github.com/0xMiden/node
    description: "Miden zkVM client-side proving rollup"
    language: "Rust"
  - url: https://github.com/ethsystems/pocs/tree/master/pocs/private-bond/privacy-l2
    description: "EthSystems PoC: private institutional bond on a privacy L2"
    language: "Noir, Solidity"
---

## Intent

Use a privacy-native rollup, either ZK-based or FHE-based, to execute financial logic with private state. This enables confidential value and unlinkable identity flows in a programmable environment, while anchoring security on Ethereum L1 via a bridge.

## Components

- Privacy rollup execution layer: either a ZK rollup that commits to state via ZK proofs of valid transitions, or an FHE rollup that computes on encrypted state.
- L1 bridge contracts: deposit, withdraw, and forced-exit paths that anchor the rollup to Ethereum.
- Client-side or coprocessor prover: generates ZK proofs for private state transitions, or orchestrates FHE computation.
- Viewing key and disclosure infrastructure: off-chain key management and attestation services for regulator access.
- Sequencer or block builder: collects and orders transactions for the rollup; centralized on most deployments today.

Shielded note logic on the rollup is described in `pattern-shielding`; disclosure flows in `pattern-regulatory-disclosure-keys-proofs`.

## Protocol

1. [user] Bridge assets from L1 to the privacy L2 via a deposit contract.
2. [contract] Assets are converted to private notes or encrypted balances on the L2.
3. [user] Execute transactions privately within the L2 (amounts, senders, receivers shielded within the rollup).
4. [prover] Generate ZK proofs (or FHE state-transition proofs) for valid state transitions.
5. [sequencer] Order and aggregate transitions; post the proof and state root to L1.
6. [contract] Optionally couple settlement with an on-chain DvP or PvP contract for atomic cross-asset flows.
7. [regulator] Access transaction details via viewing keys or disclosure proofs as required.
8. [user] Exit to L1 by submitting a withdrawal proof of valid state ownership.

## Guarantees & threat model

Guarantees:

- Private transactions: amounts, sender, and receiver are shielded within the L2.
- Private state: balances and contract state are stored as encrypted commitments or notes.
- Interoperability with L1 state: rollup state roots anchor on L1, enabling bridging and settlement with L1 assets.
- Programmable compliance: viewing keys, selective disclosure, and audit proofs can be embedded in contract logic.

Threat model:

- Bridge soundness. An exploited bridge can inflate or steal assets on L1.
- Proof system or FHE scheme soundness. Breaks here allow unauthorized state transitions.
- Sequencer liveness and honesty. A centralized or censoring sequencer can exclude transactions until the L1 forced-exit path is exercised.
- Viewing key custody. A compromised viewing key exposes the history protected under it.
- Non-standardized regulator interfaces. Each deployment rolls its own compliance hooks, creating fragmentation risk for supervisors.

## Trade-offs

- Maturity risks. Most privacy L2s are early-stage with limited production deployments.
- Migration and bridging overhead. Assets and apps must move from L1 or other L2s, adding friction and new trust surfaces.
- New attack surfaces. Viewing key compromise, bridge exploits, and sequencer censorship.
- Regulatory auditability uncertain. Selective disclosure is possible, but no standardized frameworks exist yet for regulator access or compliance proofs.
- Sequencer centralization. Most privacy L2 sequencers are currently centralized and can censor until decentralized sequencing is implemented.

## Example

- Bond issuance and secondary trading on native shielded notes.
- A private institutional payment rollup where amounts, sender, and receiver are shielded within the L2 and only state-root commitments anchor on L1.

## See also

- [Aztec documentation](https://docs.aztec.network/)
- [Miden](https://miden.xyz/)
