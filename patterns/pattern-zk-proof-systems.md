---
title: "Pattern: ZK Proof Systems"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-06-18

works-best-when:
  - Selecting a proof system for a new privacy design on Ethereum.
  - Evaluating post-quantum readiness of an existing zero-knowledge stack.
  - Comparing trust assumptions, proof size, and prover and verifier cost across backends.
avoid-when:
  - The application does not use zero-knowledge proofs.
  - A specific proof system is mandated by an existing verifier contract and no migration is planned.

context: both
context_differentiation:
  i2i: "Between institutions the dominant constraint is typically on-chain verification gas and ecosystem maturity; trusted-setup ceremonies can be run bilaterally or within a consortium. PQ migration planning is bounded by the parties' own retention windows."
  i2u: "For user-facing deployments the dominant constraint is client-side proving cost on mobile and browser targets. Transparent systems avoid toxic-waste risk and delegation-friendly backends reduce the device burden; HNDL risk for long-lived user data raises the priority of PQ-safe backends."

crops_profile:
  cr: medium
  o: yes
  p: full
  s: medium

crops_context:
  cr: "Proof systems themselves do not resist censorship; the surrounding verifier, sequencer, and DA choices do. Transparent systems eliminate the ceremony-operator trust point."
  o: "Pairing-based and hash-based proof systems have open-source reference implementations. Domain-specific zkVMs and production proving services vary in licensing."
  p: "All listed systems provide computational zero-knowledge. Metadata at the prover (proof requests, witness sizes) and at the verifier (on-chain calls, proof sizes) remains visible."
  s: "Rides on the soundness of each system plus the integrity of any trusted ceremony. Hash-based and lattice-based systems remain sound against a CRQC; pairing-based and elliptic-curve-based systems do not."

post_quantum:
  risk: high
  vector: "Pairing-based systems (Groth16, PLONK over KZG) and elliptic-curve-based systems (PLONK over IPA, Halo2) are broken by a CRQC. HNDL risk applies to any proof whose soundness must hold against a future quantum adversary, for example proofs anchoring long-lived state."
  mitigation: "Migrate to hash-based systems (STARK, hash-based SNARK) or lattice-based systems for long-horizon designs. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-shielding, pattern-noir-private-contracts, pattern-plasma-stateless-privacy, pattern-forced-withdrawal]
  see_also: [pattern-co-snark, pattern-safe-proof-delegation]

open_source_implementations:
  - url: https://github.com/arkworks-rs
    description: "arkworks ecosystem, reference Rust libraries for Groth16, PLONK, IPA"
    language: "Rust"
  - url: https://github.com/Plonky3/Plonky3
    description: "Plonky3 hash-based SNARK toolkit"
    language: "Rust"
  - url: https://github.com/starkware-libs/stone-prover
    description: "Stone STARK prover (transparent, hash-based)"
    language: "C++, Rust"
---

## Intent

Give designers a decision framework for choosing a zero-knowledge proof system on Ethereum. A zero-knowledge proof lets one party prove a statement is true without revealing the inputs. The system's commitment scheme (elliptic-curve pairings, discrete log over curves, collision-resistant hashes, or lattices) drives its post-quantum posture; the setup model drives its trust assumption; proof size and prover and verifier cost drive deployment economics.

## Components

- Pairing-based SNARKs rely on elliptic-curve pairings over curves such as BLS12-381 or BN254 and produce small constant-size proofs with low verifier cost. Groth16 requires a trusted setup per circuit; PLONK over KZG uses a universal setup.
- Elliptic-curve-based SNARKs rely on discrete log over an elliptic curve without pairings. Transparent setup, moderate proof size, medium verifier cost. PLONK over IPA and the Halo2 family sit here.
- Hash-based proof systems rely on collision-resistant hashes. STARKs over FRI and hash-based SNARKs (Plonky family, Binius) have transparent setup and remain sound against a CRQC, at the cost of larger proofs.
- Lattice-based SNARKs rely on Module-SIS or LWE hardness. Emerging, transparent, and PQ-safe; no production deployment yet.
- Hybrid systems compose a hash-based STARK with a pairing-based SNARK wrapper to achieve small on-chain proofs; the wrapper reintroduces pairing assumptions and the matching PQ exposure.

## Protocol

1. [designer] Define the statement to be proved (preimage knowledge, state-transition validity, credential verification).
2. [designer] Express the computation as an arithmetic circuit or constraint system (R1CS, PLONKish, AIR) in a chosen DSL.
3. [operator] Run the required setup: trusted per-circuit ceremony for Groth16, trusted universal ceremony for PLONK over KZG, or transparent setup for STARK, IPA, and hash-based systems.
4. [prover] Generate a proof from the private witness and the public inputs.
5. [verifier] Check the proof on-chain or off-chain against the public statement; verifier cost is driven by proof size and the verification algorithm.
6. [designer] Re-evaluate the backend choice on roadmap checkpoints, driven by PQ migration deadlines and proof-size or gas improvements.

## Proof system comparison

| System            | Trust setup           | PQ-safe          | Proof size | Prover cost | Verifier cost | Used by                                                                                                |
| ----------------- | --------------------- | ---------------- | ---------- | ----------- | ------------- | ------------------------------------------------------------------------------------------------------ |
| Groth16           | Trusted per circuit   | No (pairings)    | ~200 B     | Low         | Low           | [Railgun](../vendors/railgun.md), [EY](../vendors/ey.md), [Privacy Pools](../vendors/privacypools.md) |
| PLONK over KZG    | Trusted universal     | No (pairings)    | ~400 B     | Medium      | Low           | [Aztec](../vendors/aztec.md), [zkSync](../vendors/zksync.md)                                          |
| PLONK over IPA    | Transparent           | No (EC)          | ~1 KB      | Medium      | Medium        | ZCash                                                                                                  |
| STARK over FRI    | Transparent           | Yes (hash-based) | ~50-200 KB | High        | Medium        | [Miden](../vendors/miden.md)                                                                           |
| Hash-based SNARKs | Transparent           | Yes (hash-based) | ~70-250 KB | High        | Medium        | Plonky3, Binius                                                                                        |
| Lattice-based     | Transparent           | Yes (lattices)   | TBD        | TBD         | TBD           | Research stage (Latticefold, Espalier)                                                                 |

Benchmarks for Ethereum block-proving workloads are available at [ethproofs.org CSP benchmarks](https://ethproofs.org/csp-benchmarks). The table above reflects typical privacy-application proof characteristics; block-proving benchmarks differ in scale.

### Key dimensions

- Trust setup: trusted setups carry toxic-waste risk; transparent systems eliminate it. For institutional adoption transparent is generally preferred.
- PQ safety: pairing-based and elliptic-curve-based systems are broken by a CRQC (Shor's algorithm). Hash-based and lattice-based systems survive. Grover weakens but does not break hash-based soundness; parameter sizes can be raised to compensate.
- Proof size vs verification cost: pairing-based SNARKs produce compact proofs that are cheap to verify on-chain. Hash-based systems produce larger proofs that are improving via recursive composition and proof compression.
- Prover cost: hash-based systems are more expensive to prove but align well with GPU acceleration thanks to NTT-based arithmetic, a useful property for client-side proving.

## Guarantees & threat model

Guarantees:

- Completeness: an honest prover with a valid witness always convinces the verifier.
- Soundness: all listed systems offer computational soundness; a cheating prover cannot convince the verifier except with negligible probability under the stated cryptographic assumption.
- Zero-knowledge: each system hides the prover's private inputs from the verifier.
- Transparency (STARK, IPA, hash-based SNARK): no trusted party or ceremony is required.
- PQ safety: hash-based and lattice-based systems remain sound against quantum adversaries; pairing-based and elliptic-curve-based systems do not.

Threat model:

- Soundness holds only under the stated assumption: hardness of discrete log for EC-based systems, pairing-related assumptions for pairing-based systems, collision resistance for hash-based systems, Module-SIS or LWE for lattice-based systems.
- Trusted setups require that at least one ceremony participant was honest and that toxic waste was destroyed; a compromised ceremony allows proofs for false statements.
- Side-channel attacks on the prover or the signer for recursion are out of scope.
- Verifier contract bugs or mis-wired public inputs can accept proofs that the proof system itself rejects.

## Trade-offs

- PQ migration cost: existing pairing-based circuits must be re-implemented for hash-based or lattice-based backends; constraint systems differ (R1CS vs AIR) and field arithmetic differs.
- Proof size on-chain: hash-based proofs are roughly 100 to 1000 times larger than pairing-based proofs. EIP-4844 blobs help but on-chain verification remains more expensive.
- Hybrid complexity: wrapping a STARK in a pairing-based SNARK yields small on-chain proofs but reintroduces pairing assumptions and the matching PQ exposure.
- Ecosystem maturity: pairing-based tooling (Circom, Noir, Halo2) is more mature than STARK tooling, though the gap is closing.

## Example

A privacy L2 uses pairing-based PLONK for transaction proofs today. To prepare for post-quantum migration, the team evaluates moving to a FRI-based STARK backend. Proof size grows from about 400 bytes to about 100 kilobytes, but proofs become PQ-safe and require no trusted setup. Recursive composition keeps on-chain verification cost manageable, and EIP-4844 blobs carry proof Data Availability at lower cost than calldata.

## See also

- [Post-Quantum Threats](../domains/post-quantum.md)
- [Collaborative zk-SNARKs (Ozdemir & Boneh, 2021)](https://eprint.iacr.org/2021/1530.pdf)
- [EthProofs CSP benchmarks](https://ethproofs.org/csp-benchmarks)
- [Espalier: succinct lattice arguments for client-side proving (Palmette, 2026)](https://palmette.xyz/espalier.pdf): v0.9 preprint, no public implementation, prover figures self-reported
