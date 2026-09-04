---
title: "Pattern: Relay-Mediated Proving"
status: ready
maturity: concept
layer: offchain
last_reviewed: 2026-06-18

works-best-when:
  - Client cannot run a SNARK prover (constrained hardware, low power, intermittent connectivity)
  - Multiple relays are available so the client can fan out
  - The application contract can accept a public input pinning the submitter address
avoid-when:
  - Client can prove locally; use direct submission instead
  - Proofs must be portable (e.g., minted as transferable artifacts)
  - Front-running the proof would not extract value (no incentive for relay theft)

context: both
context_differentiation:
  i2i: "Institutional client signs an internal request; proving outsourced to an MPC custodian or shared prover service."
  i2u: "User device signs a portable message; relay generates the SNARK and submits on the user's behalf."

crops_profile:
  cr: medium
  o: yes
  p: partial
  s: medium

crops_context:
  cr: "Relay-set diversity is the gating factor. Single-relay deployments are coercible. N >= 8 across multiple jurisdictions is a baseline."
  o: "Open-source proof systems and signature gadgets exist (Noir, Halo2, Plonky2). Relay software is application-specific."
  p: "Client signing data hidden from the chain. Relay sees the signed message in cleartext during proof construction."
  s: "Submitter binding closes proof-stealing front-running. The relay remains a privacy boundary; degrades to low if the relay set is small or compelled wholesale."

post_quantum:
  risk: high
  vector: "ECDSA / EdDSA signatures and pairing-based SNARKs (Groth16, KZG-PLONK) broken by CRQC"
  mitigation: "Post-quantum signatures (Falcon, Dilithium) with STARK or hash-based SNARKs"

standards: [FIPS 186-5, RFC 6979]

related_patterns:
  composes_with: [pattern-mesh-store-forward-submission, pattern-forward-secure-signatures, pattern-network-anonymity, pattern-recipient-derived-receive-addresses]
  alternative_to: [pattern-safe-proof-delegation]
  see_also: [pattern-permissionless-spend-auth, pattern-private-mtp-auth]

open_source_implementations:
  - url: https://noir-lang.org/docs/noir/standard_library/cryptographic_primitives
    description: "Noir stdlib ECDSA-secp256k1 in-circuit verifier"
    language: Noir
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec packages including Barretenberg UltraHonk prover"
    language: C++/Rust
  - url: https://github.com/0xPolygonZero/plonky2
    description: "Plonky2: recursive SNARK with FRI"
    language: Rust
---

## Intent

Split a typical zero-knowledge proof flow into two roles. The client signs a portable signed message offline. A relay generates the SNARK over that message and submits it on-chain. To prevent a front-runner from lifting the proof off the mempool and re-submitting it from their own address, the proof binds to the relay's submitter address as a public input that the application contract checks against `msg.sender`.

The pattern is distinct from `pattern-safe-proof-delegation.md`, which addresses intent-based delegation in a wallet UX context (the client *could* prove locally but chooses to delegate). Relay-mediated proving is for clients that cannot prove locally at all.

## Components

- Client-side signing primitive: a signature the client device already supports (ECDSA-secp256k1 with RFC 6979 plus canonical-s, EdDSA, or RSA-PSS). The client never holds witness data the relay can extract beyond what the signed message carries.
- In-circuit signature verification: Noir `std::ecdsa_secp256k1`, in-circuit EdDSA, or equivalent gadget. The circuit recomputes the signed-message encoding bit-identically and verifies against the client's public key.
- Submitter binding: the proof's public inputs include a `submitter` field. The circuit asserts the field equals a witness-shared value; the application contract on-chain asserts `publicInputs.submitter == msg.sender`. Together these close proof-stealing front-running.
- Relay set: multiple relays per deployment. Clients fan out to `k < N` relays. Single-relay deployments do not satisfy the privacy assumption.

## Protocol

1. [client] Produce `(payload, signature)` where `payload` includes the application context. The signed-message encoding is pinned bit-identically to the in-circuit verifier's encoding.
2. [client] Encrypt the signed message to the chosen relay's current decryption key under IND-CCA2 AEAD; transport via online IP, mesh, or courier.
3. [relay] Decrypt, verify the signature against expected public-key sources, and check application preconditions.
4. [relay] Construct the witness and invoke the prover (Noir + UltraHonk, Halo2, Plonky2). Public inputs include the submitter binding field set to the relay's submission EOA.
5. [relay] Submit on-chain through anonymous transport. The application contract verifies the proof, asserts `publicInputs.submitter == msg.sender`, then executes application logic.
6. [relay] Rotate submission EOAs on a published cadence (e.g., per 24 hours), funding fresh EOAs through anonymity-preserving paths to avoid long-lived submission-funding trails.

## Guarantees & threat model

- Front-running resistance: a front-runner who lifts the proof from the mempool cannot re-submit from a different address; the proof reverts on a different `msg.sender`.
- Witness confidentiality from the chain: only public inputs are revealed on-chain. Private inputs (signature, membership path, internal state) stay with the relay during proof generation.
- No client-prover requirement: the client device runs only its native signature primitive.
- Threat model: adversaries include front-runners, the chain itself, and a non-quorum of compelled relays. Out of scope: a fully compelled or breached relay set; signed-message-encoding mismatches between client and circuit (a known historical bug class).

## Trade-offs

- The relay sees the signed message in cleartext. Mitigations: client fan-out across relays, relay-set jurisdictional diversity, decryption-key rotation.
- Pinned signed-message encoding. Client-side serialization and circuit-side recomputation must match byte-for-byte. Audit must verify equality.
- Witness-binding correctness. All circuit gadgets that consume the client's public key (signature verification, membership leaf hash, nullifier hash) MUST share a single witness variable. Independent equality constraints have shipped historically as a soundness bug.
- Submitter binding precludes transferable proofs. Applications that mint transferable proof artifacts need a different proof-stealing mitigation.
- Relay-economic recovery is open. Relays pay gas for submitted (and reverted) transactions. Deployment-level compensation models are out of scope.

## Example

A privacy-preserving token airdrop with offline eligibility credentials. Eligibility is pre-distributed through a tamper-resistant medium (paper QR with embedded signing key, low-power hardware token). To redeem, the recipient signs a redemption message and forwards it to one of N airdrop-relay operators. The relay generates an UltraHonk proof of eligibility-Merkle-tree membership and signature validity, with `publicInputs.relaySubmitter` set to the relay's rotating submission EOA. The airdrop contract verifies the proof and asserts `msg.sender == publicInputs.relaySubmitter`. A front-runner who lifts the proof cannot redirect the airdrop because the submitter check fails.

## See also

- [Noir cryptographic primitives](https://noir-lang.org/docs/noir/standard_library/cryptographic_primitives).
- [Barretenberg UltraHonk](https://github.com/AztecProtocol/barretenberg).
- [Plonky2](https://github.com/0xPolygonZero/plonky2).
- [RFC 6979: Deterministic Usage of DSA and ECDSA](https://www.rfc-editor.org/rfc/rfc6979).
