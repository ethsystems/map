---
title: "Pattern: Confidential Policy Verdicts"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-08-02

works-best-when:
  - A standing ruleset governs many actors who never individually signed it.
  - Publishing the ruleset would teach an adversary how to evade it.
  - Enforcement must happen before execution rather than as an after-the-fact attestation.
avoid-when:
  - The principal can pre-authorise each action, where a signed mandate is simpler.
  - The policy is public anyway, where an ordinary on-chain allowlist suffices.
  - The action itself needs hiding, which this pattern does not provide.

context: i2i

crops_profile:
  cr: low
  o: yes
  p: partial
  s: medium

crops_context:
  cr: "Denial is the pattern's function, so it is a censorship mechanism by construction. It stays at `low` rather than `none` because the Guard bounds one execution path and the principal retains whatever authority it holds outside that path. A domain operator can refuse any action without stating a reason, and the proof system gives the agent no recourse."
  o: "The standard is an open ERC with a CC0 reference implementation, and the proving stack (Noir, UltraHonk, SP1) is open source. Anyone can run a verifier and check a domain's registered root."
  p: "The pattern hides the policy, not the action. A permitted action executes on a public chain and is fully visible. What no observer learns, including the executing agent, is the rules that permitted it. A denied action is never submitted, which is a side effect rather than a guarantee."
  s: "Correctness rests on the proving system and on the Guard recomputing `actionCommitment` itself. The reference implementation is unaudited, and the ERC-165 `interfaceId` is still a placeholder pending finalisation."

post_quantum:
  risk: high
  vector: "UltraHonk verification is pairing-based over BN254 with KZG commitments, which a CRQC breaks. Forged proofs would let an attacker manufacture allow verdicts against any registered domain. Policy commitments themselves are hash-based and hold up better than the proof system that opens them."
  mitigation: "Move the proving program to a hash-based system with no trusted setup. The reference implementation already carries an SP1 program alongside the Noir circuit, which is the practical migration route. See [Post-Quantum Threats](../domains/post-quantum.md)."

visibility:
  counterparty: []
  chain: [agentId, domainId, policyRoot, actionCommitment, nullifier]
  regulator: [ruleset via the domain registrar]
  public: [that some committed policy permitted this action]

standards: [ERC-8354, ERC-7812, ERC-8004, ERC-165]

related_patterns:
  composes_with: [pattern-verifiable-attestation]
  see_also: [pattern-regulatory-disclosure-keys-proofs, pattern-commit-and-prove]

open_source_implementations:
  - url: https://github.com/zexoverz/confidential-agent-policy-verdicts
    description: "ERC-8354 reference implementation, Noir circuit with generated UltraHonk verifier (unaudited, not deployed)"
    language: "Solidity / Noir"
---

## Intent

Gate an autonomous agent's action on a policy the agent is not allowed to read. A verifier learns that some committed ruleset was evaluated and returned allow, and learns nothing about the ruleset's contents.

## Components

- Policy Domain: an operator that maintains a private ruleset and runs an off-chain policy engine.
- Policy Commitment: a hash of the ruleset, registered as a statement in an ERC-7812 Evidence Registry.
- Policy Root: the sparse Merkle tree root containing the current commitment, which the proof is evaluated against.
- Verdict envelope: agent identity, domain, policy root, action commitment, executor, expiry, nullifier, and decision. Every field is a public input.
- Guard: the on-chain contract that verifies a verdict and gates execution on it.
- Agent identity: an ERC-8004 Identity Registry token, which supplies the subject the verdict binds to.

## Protocol

1. [operator] Register a domain, commit the ruleset hash to the ERC-7812 registry, and publish the verifier address and program key.
2. [agent] Propose an action and request a verdict from the domain's policy engine.
3. [prover] Evaluate the action against the private ruleset and produce a proof binding the decision to the agent, the policy root, a commitment to the action, the permitted executor, an expiry, and a single-use nullifier.
4. [contract] Recompute `actionCommitment` from the call about to execute, and reject any commitment supplied by the caller.
5. [contract] Check that the domain is active, the decision is allow, the executor is the caller, the verdict has not expired, the nullifier is unburned, and the policy root is acceptable.
6. [contract] Verify the proof against the domain's program key, burn the nullifier, emit `VerdictConsumed`, and execute.
7. [operator] Rotate the root as rules change, accepting superseded roots for a bounded window, with revocation taking effect immediately.

## Guarantees & threat model

- Hides the ruleset from the chain, from relying parties, and from the agent being governed.
- Binds one verdict to one call through `actionCommitment`, so a verdict cannot be replayed against a different action.
- The executor check stops a third party observing a pending verdict and consuming it ahead of the intended executor.
- Threat model: the operator is trusted to author honest rules. A proof shows the ruleset was evaluated correctly, never that the ruleset is reasonable. A hostile operator can deny arbitrarily, or commit to a policy that permits everything, and no verifier can tell.
- Root rotation leaves a window in which a superseded policy still authorises actions.
- Out of scope: hiding the action, hiding which domain gated it, and hiding that a verdict was consumed.

## Trade-offs

- Proving happens off-chain per action and adds latency to every gated call.
- On-chain cost is one proof verification plus a nullifier write.
- Denied actions never reach the chain, so a watcher who knows an agent was about to act can infer denial from silence.
- The `interfaceId` is a placeholder until the ERC leaves Draft, so ERC-165 detection is not yet stable.
- Two hash functions coexist, keccak256 for on-chain digests and Poseidon in-circuit, which is a recurring source of commitment mismatch bugs.

## Example

- A corporate card issuer maintains fraud rules that are deliberately unpublished, because a published rule is an evasion guide.
- A procurement agent proposes a payment. The issuer's engine evaluates it against the committed ruleset and returns an allow verdict with a proof.
- The Guard recomputes the action commitment, checks the envelope, verifies the proof, and releases the payment.
- Merchant, agent, and chain observers see that a payment was permitted under domain `X` at root `R`. None of them learn a single rule.
- Implementation status: a Noir allowlist circuit and generated UltraHonk verifier verify a real proof through `consume` in an EVM test harness, across 23 passing Foundry tests. Nothing is deployed to a public network, and the contracts are unaudited.

## See also

- [ERC-8354 pull request (ethereum/ERCs#1919)](https://github.com/ethereum/ERCs/pull/1919)
- [ERC-8354 discussion on Ethereum Magicians](https://ethereum-magicians.org/t/erc-8354-confidential-agent-policy-verdicts/29088)
- [ERC-7812: ZK Identity Registry](https://eips.ethereum.org/EIPS/eip-7812)
- [ERC-8004: Trustless Agents](https://eips.ethereum.org/EIPS/eip-8004)
