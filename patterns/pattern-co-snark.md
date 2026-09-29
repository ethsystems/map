---
title: "Pattern: Delegated Proving (co-SNARKs)"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-09-29

works-best-when:
  - A user or institution lacks the compute, memory, or battery to generate a zero-knowledge proof client-side and wants to offload the work without disclosing the witness.
  - The delegatee set can be run by independent operators so no single party sees the full witness.
avoid-when:
  - Client-side proving is feasible and low-latency is critical.
  - Only one proving node is available (no honest-majority distribution possible); use a TEE-based prover instead.

context: both
context_differentiation:
  i2i: "Between institutions, delegated proving is typically contracted between known parties with SLAs and legal recourse. The prover set can be small and stable. Honest-majority risk is mitigated by bilateral agreements and audit logs."
  i2u: "For users, the prover set must be permissionless or at least operator-diverse so that no coalition can recover the witness. Economic bonding with slashing and publicly auditable proving logs protect against collusion. A user has no recourse if the prover set silently leaks their witness."

crops_profile:
  cr: medium
  o: yes
  p: full
  s: medium

crops_context:
  cr: "Reaches `high` when the prover network is permissionless and bond-backed with slashing for Byzantine behaviour. Drops to `low` when a single proving service controls the pipeline."
  o: "Proving-framework implementations are open source (TACEO co-snarks: MIT/Apache-2.0, with the co-circom components under GPL-3.0); production deployments may bundle proprietary orchestration."
  p: "The witness stays hidden from each individual prover and from the verifier. Metadata about who requested a proof, when, and against which circuit can still leak."
  s: "Rides on the soundness of the underlying SNARK and the corruption threshold of the MPC protocol (honest majority for GSZ, replicated, or Shamir sharing; one honest node for SPDZ-style protocols). Trusted-setup requirements inherit from the SNARK (Groth16 needs per-circuit setup; PLONK/KZG uses universal setup)."

post_quantum:
  risk: high
  vector: "Pairing-based SNARKs (Groth16, PLONK/KZG) broken by CRQC. MPC communication inherits the underlying key-exchange assumptions."
  mitigation: "Collaborative versions of hash-based proof systems exist only as research (Ozdemir & Boneh adapt Fractal, at N-times proof size and verification cost). See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-shielding, pattern-safe-proof-delegation, pattern-permissionless-spend-auth]
  alternative_to: [pattern-tee-based-privacy]
  see_also: [pattern-private-shared-state-cosnark, pattern-zk-proof-systems]

open_source_implementations:
  - url: https://github.com/TaceoLabs/co-snarks
    description: "co-SNARK proving framework: Groth16 and PLONK (coCircom), UltraHonk (coNoir); README marks it experimental and un-audited"
    language: "Rust"
---

## Intent

Offload zero-knowledge proof generation to a distributed prover network without revealing the witness. The user secret-shares their witness across several proving nodes; the nodes jointly run an MPC protocol to compute a single SNARK proof; no individual node ever reconstructs the full witness. The resulting proof is indistinguishable from one produced client-side and is verified on-chain or off-chain with no changes on the verifier side.

This pattern covers delegated proving for a single prover's witness. For multi-party joint computation over shared secret inputs (e.g. a consortium ledger), see `pattern-private-shared-state-cosnark`.

## Components

- User or application holds the witness and wants a proof generated without exposing the witness.
- Share-distribution layer splits the witness using secret-sharing (additive, replicated, or Shamir) and routes shares to proving nodes.
- Distributed prover network runs the MPC protocol to jointly compute the SNARK. Each node sees only its share.
- Coordinator sequences MPC rounds and assembles the final proof. Can be one of the proving nodes or a separate role.
- Verifier checks the final proof exactly as it would check a client-side SNARK. No changes on the verification side.

## Protocol

1. [user] Secret-share the witness across N proving nodes.
2. [prover] Proving nodes jointly execute the co-SNARK MPC protocol, exchanging shares across multiple rounds to compute witness polynomials and commitments.
3. [prover] The coordinator assembles the final proof from the MPC output.
4. [user] Receive the assembled proof from the coordinator.
5. [contract] A verifier contract (or off-chain verifier) checks the proof against the public statement.

## Guarantees & threat model

Guarantees:

- The witness stays hidden from every individual prover and from the verifier.
- Verification cost is identical to a client-side SNARK for the same circuit.
- Preserves trade secrets, user balances, or model weights while fewer nodes collude than the MPC protocol tolerates.

Threat model:

- Soundness of the underlying SNARK, including any trusted-setup ceremony.
- Corruption threshold of the MPC protocol: honest majority for GSZ, replicated, or Shamir sharing (TACEO co-snarks uses 3-party replicated and Shamir sharing), or one honest node for SPDZ. A coalition above the threshold can recover the witness. It cannot forge proofs of false statements: knowledge soundness comes from the underlying SNARK even if all provers collude.
- Non-censoring coordinator. A malicious coordinator can refuse to finalize or selectively drop requests.
- Authenticated and confidential channels between nodes. Metadata about participation and timing is out of scope.

## Trade-offs

- Communication overhead. Bandwidth grows with circuit size and the number of provers; round count is sub-linear in constraint count. PLONK-style provers need more communication than Groth16.
- New infrastructure requirements: MPC nodes, share routing, key management.
- Liveness depends on all designated nodes remaining online through the proving session. Dropouts typically force a restart.
- Latency is higher than a single prover on the same hardware for small circuits, where MPC round trips dominate. Ozdemir & Boneh measure near-single-prover runtime for large circuits with honest-majority GSZ over 3 Gb/s links, 2x for SPDZ, and slowdown growing on low-bandwidth links. Not suitable for sub-second proving budgets.

## Example

- A user holds a shielded balance and wants to prove a spend on a mobile device. The wallet secret-shares the spending witness across three independent proving operators. The operators jointly produce a Groth16 proof. None of them sees the balance or the spend amount. The user submits the proof to the shielded pool contract.

## See also

- [Collaborative zk-SNARKs (Ozdemir & Boneh, USENIX Security 2022)](https://eprint.iacr.org/2021/1530.pdf)
- [TACEO private proof delegation](https://core.taceo.io/articles/private-proof-delegation/)
- [TACEO Merces vendor page](../vendors/taceo-merces.md)
