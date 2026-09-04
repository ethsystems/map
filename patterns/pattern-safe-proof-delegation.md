---
title: "Pattern: Safe Proof Delegation"
status: ready
maturity: testnet
type: standard
layer: L1
last_reviewed: 2026-06-18

works-best-when:
  - Users cannot generate zero-knowledge proofs locally (mobile wallets, hardware wallets, constrained devices).
  - A server-side prover or privacy-aware RPC bridges the gap before native wallet support.
  - Delegation must be revocable without moving funds to new notes or a new address.
avoid-when:
  - Client-side proving is already available and performant for the target workload.
  - The prover seeing transaction details is unacceptable even temporarily; prefer a distributed or TEE-hosted prover instead.

context: both
context_differentiation:
  i2i: "Between institutions, proof delegation typically happens under bilateral contracts with SLAs and audit rights. The authorising institution can demand operational logs and, through legal recourse, punish deviation even though deviation cannot be prevented cryptographically."
  i2u: "For users, the prover is usually an external service. The pattern must assume the prover can be adversarial at any moment: output-secret rotation and short-lived intents are how the user retains the ability to unplug a compromised prover without moving funds."

crops_profile:
  cr: low
  o: yes
  p: partial
  s: high

crops_context:
  cr: "Depends on prover liveness. A single prover service is a censorship surface; CR approaches `medium` when multiple interchangeable provers accept the same intent format and the user can switch without re-sealing state."
  o: "Intent schema, circuits, and verifier contracts are open source; provers can be hosted by anyone implementing the schema."
  p: "The prover sees transaction plaintext (amount, recipient, token). The verifier and the chain see only the proof. Combining with a TEE-hosted or MPC-based prover can hide plaintext from the prover operator as well."
  s: "Funds cannot be stolen or redirected: the signed intent binds every material parameter. Security rides on the soundness of the zero-knowledge proof system, correct intent-schema binding, and rotatable output secrets that unlink future transactions from a past prover."

post_quantum:
  risk: high
  vector: "Pairing-based proof systems and ECDSA/Schnorr intent signatures are broken by a CRQC. HNDL risk applies: an attacker recording today's intent signatures could forge future spends once quantum capability is available."
  mitigation: "Post-quantum signature schemes (ML-DSA, SLH-DSA) for the intent layer and STARK-based circuits for the proof system. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [EIP-8182]

related_patterns:
  requires: [pattern-shielding, pattern-zk-proof-systems]
  composes_with: [pattern-permissionless-spend-auth, pattern-co-snark, pattern-tee-based-privacy]
  see_also: [pattern-user-controlled-viewing-keys]

open_source_implementations:
  - url: https://eips.ethereum.org/EIPS/eip-8182
    description: "EIP-8182 (Private ETH and ERC-20 Transfers): canonical intent digest and rotatable output-secret protocol"
    language: "specification"
---

## Intent

Let a user delegate zero-knowledge proof generation to an external prover (a privacy RPC, a hardware accelerator, or a third-party service) without giving that prover the ability to forge, redirect, or overspend. The user signs a canonical intent digest that binds every material parameter; the prover can produce a valid proof that executes exactly that intent and nothing else.

## Components

- Canonical intent digest that binds operation kind, token, recipient, amount, nonce, expiry, chain identifier, and any required additional fields such as policy version and pool address.
- Two protocol secrets: an immutable spending key that controls nullifier derivation, and a rotatable output secret that controls deterministic output randomness. Note-delivery encryption is handled by companion standards.
- Intent nullifier set, distinct from the note nullifier set, that prevents replay of a signed intent.
- Zero-knowledge circuit that checks the signature against the authorising address, verifies that every public input matches the intent digest, validates note openings and Merkle paths, and enforces value conservation.

## Protocol

1. [user] Compute the canonical intent digest and sign it inside the wallet; the signed intent is never broadcast.
2. [user] Send the signed intent plus witness data (note openings, Merkle paths, keys) to the prover over an encrypted channel.
3. [prover] Run the circuit to produce a proof that binds the witness to the signed intent.
4. [prover] Submit the proof and public inputs to the pool contract.
5. [contract] Verify the proof, consume note nullifiers, and record the intent nullifier to prevent replay.
6. [user] When switching provers, rotate the output secret so the former prover can no longer derive output randomness for future transactions; existing notes remain spendable because the spending key is unchanged.

## Guarantees & threat model

Guarantees:

- No custody: the prover never holds keys that can unilaterally spend. Every spend requires a user-signed intent.
- Tamper-proof intents: modifying any bound parameter invalidates the proof.
- Replay prevention: each signed intent can execute exactly once via the intent nullifier set.
- Revocable delegation: rotating the output secret cuts off a former prover's ability to derive future output randomness without moving funds.

Threat model:

- Soundness of the underlying proof system.
- Correctness of the intent-digest schema and domain separation; a bug that lets the prover substitute a public input breaks the tamper-proof guarantee.
- Confidentiality of the note-delivery channel. A malicious prover can emit unusable delivery payloads, rendering outputs unrecoverable by the recipient (funds are not lost for the sender, but the recipient cannot claim them).
- Liveness of at least one prover; a single offline prover stalls the user until they switch.
- Microarchitectural or supply-chain attacks on the prover that leak plaintext are in scope for this pattern but are mitigated by combining with a TEE-hosted or MPC-based prover.

## Trade-offs

- The prover sees transaction plaintext. Privacy from the prover operator itself requires a TEE-based or distributed prover, at the cost of more infrastructure.
- Liveness depends on the prover. If the prover is offline, the user must switch or move to client-side proving.
- Expiry tuning matters: too short and transactions fail before submission, too long and a compromised prover has more room to manoeuvre within the fixed intent.
- Output-secret rotation has an operational cost: the user must retain the prior secret until the stale-root window expires and authorised transactions have settled.

## Example

A user holds shielded balances in a mobile wallet that cannot generate zero-knowledge proofs. The wallet signs an intent to send 100 units to a specific recipient with a ten-minute expiry and forwards the signed intent plus witness data to a privacy RPC. The RPC generates the proof and submits it on-chain within three minutes. The RPC sees the transfer details but cannot redirect funds. When the user later switches RPC providers, they rotate the output secret; the former RPC can no longer derive output randomness for future transactions, and existing notes remain spendable.

## See also

- [EIP-8182: Private ETH and ERC-20 Transfers](https://eips.ethereum.org/EIPS/eip-8182)
- [Railgun](../vendors/railgun.md)
- [Espalier: succinct lattice arguments for client-side proving (Palmette, 2026)](https://palmette.xyz/espalier.pdf): targets on-device proving that would reduce the need for delegation, but is a preprint with no public implementation
