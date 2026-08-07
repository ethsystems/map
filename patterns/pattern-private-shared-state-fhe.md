---
title: "Pattern: Private Shared State (FHE)"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-06-18

works-best-when:
  - Multiple institutions share a ledger, pool, or order book and must hide individual positions from each other.
  - Computation on encrypted state must happen without decrypting intermediate values.
  - Higher batched throughput is needed than MPC-plus-ZK approaches can offer.
avoid-when:
  - Single-party privacy is sufficient (use shielding instead).
  - Sub-second latency is critical (Fully Homomorphic Encryption overhead is high).
  - No tolerance for a threshold decryption committee in the trust model.

context: both
context_differentiation:
  i2i: "The threshold decryption committee and the Fully Homomorphic Encryption coprocessor are typically operated by consortium members or a neutral third party under SLA. Committee collusion risk is bounded by contractual controls and committee diversity; institutions can enforce geographic and legal separation."
  i2u: "For user-facing deployments the threshold committee must be operator-diverse so that no coalition can decrypt user state. A user has no recourse if the committee silently releases decryption material. Committee rotation, transparent key ceremonies, and bond-backed participation matter more than in the i2i setting."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Protocol-level participation is open but requires access to a Fully Homomorphic Encryption coprocessor and the decryption committee. Drops to `low` when a single operator controls both. Reaches `high` only when the coprocessor and committee are permissionless or bond-backed."
  o: "Reference Fully Homomorphic Encryption libraries are open source under permissive licenses. Production coprocessors often bundle proprietary optimisations, orchestration, and hosted decryption services."
  p: "No plaintext is exposed during computation, including to the coprocessor operator, under the threshold-committee assumption. A breach of the committee above the threshold leaks all encrypted state; metadata about callers and timing remains visible on-chain."
  s: "Rides on the hardness assumptions of the Fully Homomorphic Encryption scheme (typically LWE-based), correct distributed key generation, and honest-threshold behaviour of the decryption committee."

post_quantum:
  risk: low
  vector: "Lattice-based Fully Homomorphic Encryption schemes (TFHE, BFV, CKKS) rely on LWE, which is not broken by known quantum algorithms. Side-channel risk and parameter choice still matter."
  mitigation: "Track NIST-selected lattice parameter sets and any quantum-side-channel guidance. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-stealth-addresses, pattern-regulatory-disclosure-keys-proofs, pattern-threshold-encrypted-mempool]
  alternative_to: [pattern-private-shared-state-cosnark, pattern-private-shared-state-tee]
  see_also: [pattern-shielding, pattern-co-snark, pattern-private-set-intersection-fhe]

open_source_implementations:
  - url: https://github.com/zama-ai/fhevm
    description: "Zama fhEVM, Fully Homomorphic Encryption runtime for EVM contracts (mainnet)"
    language: "Solidity, Rust"
  - url: https://github.com/zama-ai/tfhe-rs
    description: "TFHE-rs, pure-Rust implementation of TFHE primitives used by coprocessors"
    language: Rust
---

## Intent

Enable N parties to jointly read and write shared on-chain state (balances, positions, order books, collateral pools) while keeping each party's individual data private from the others and from the infrastructure operator. This variant stores state encrypted on-chain under a shared Fully Homomorphic Encryption key, runs state transitions as homomorphic operations on ciphertexts, and controls reads through a threshold decryption committee.

Unlike single-party shielding, private shared state requires computation across multiple parties' secrets.

## Components

- Fully Homomorphic Encryption scheme (for example TFHE or BFV) supporting the target state-transition circuit on encrypted data.
- Shared public key generated via distributed key generation by a threshold decryption committee; the matching secret is never assembled in one place.
- Fully Homomorphic Encryption coprocessor evaluates state transitions on ciphertexts and emits updated encrypted state plus a correctness proof.
- Threshold decryption committee jointly decrypts scoped read requests for participants or auditors.
- On-chain state store holds encrypted balances and commitments; a verifier contract anchors the correctness proof.
- Regulatory disclosure path issues threshold-controlled decryption keys scoped to a specific position or time window.

## Protocol

1. [user] The threshold committee runs a distributed key generation and publishes the shared Fully Homomorphic Encryption public key.
2. [user] Each party deposits assets; balances are encrypted under the shared key and posted as on-chain ciphertexts.
3. [user] A party submits an encrypted state-transition request (transfer, trade, margin call).
4. [prover] The coprocessor evaluates the transition homomorphically on ciphertexts and produces the updated encrypted state plus a correctness proof.
5. [contract] The verifier contract checks the proof and commits the new encrypted state root.
6. [auditor] Regulators or participants request scoped reads; the committee performs threshold decryption bound to the approved scope.

## Guarantees & threat model

Guarantees:

- No plaintext is exposed during computation, including to the coprocessor operator, under the threshold-committee assumption.
- State correctness is enforced by the coprocessor's correctness proof verified on-chain.
- Settlement finality anchored to Ethereum L1 or an L2.
- Scoped auditability via threshold-controlled decryption for specific positions or windows.

Threat model:

- Hardness of the underlying lattice assumption and correct parameter choice.
- Honest-threshold behaviour of the decryption committee. A breach above the threshold leaks all encrypted state bulk-decryption capability.
- Coprocessor correctness. A malicious coprocessor without a correctness proof could post invalid state; the proof-verifier contract mitigates this.
- Metadata about callers, contract addresses, and timing remains visible on-chain.
- Sender and receiver addresses are not hidden by default; address unlinkability requires composition with stealth addresses.

## Trade-offs

- Fully Homomorphic Encryption computation overhead is high per operation; batching and coprocessor-level optimisations are required to reach useful throughput.
- The threshold committee must be online for any read; offline committee degrades to unavailability, not privacy loss.
- Ciphertext size and on-chain storage cost are larger than plaintext or commitment-only state.
- Tooling and developer experience are less mature than Solidity with plaintext state; constrained language subsets and specific coprocessor APIs apply.

## Example

- Three banks share a tokenised-bond collateral pool on an Ethereum L2 with Fully Homomorphic Encryption support. Each bank's deposit is encrypted under the shared key; on-chain state is fully ciphertext. A margin call triggers the coprocessor to evaluate aggregate collateral coverage homomorphically and emit the updated encrypted state. The regulator uses a threshold-controlled decryption key to audit one bank's position without learning the others.

## See also

- [Zama](../vendors/zama.md)
- [Fhenix](../vendors/fhenix.md)
- [Zama Protocol documentation (FHE on blockchain)](https://docs.zama.org/protocol/protocol/overview)
