---
title: "Pattern: Publicly Verifiable DKG & Threshold Decryption"
status: draft
maturity: testnet
type: standard
layer: offchain
last_reviewed: 2026-07-21

works-best-when:
  - Decryption authority must be distributed across independent operators so no single party can unilaterally expose private inputs.
  - Every cryptographic operation (key generation, share publication, decryption) must be publicly verifiable on-chain.
  - Misbehavior by committee members must be cryptographically detectable and provable for slashing or dispute resolution.
avoid-when:
  - A single trusted key holder is acceptable — use simpler asymmetric encryption.
  - On-chain verification cost (gas) is prohibitive for the target deployment.
  - The committee is small and fully trusted; PVSS and on-chain verification add complexity without proportional benefit.

context: both
context_differentiation:
  i2i: "Institutions gain auditable proof that no single operator could have decrypted inputs or outputs prematurely. On-chain verification artefacts serve as compliance records and shared evidence in bilateral disputes."
  i2u: "End users are protected by the same k-of-n threshold as institutions: no single node can decrypt their input, and any attempt is provable on-chain. The guarantee degrades to that of a single trusted key holder if the operator set is captured."

crops_profile:
  cr: high
  o: yes
  p: full
  s: high

crops_context:
  cr: "No single operator holds sufficient key material to decrypt; censorship requires coordinated action across a threshold of independent nodes. Verifiable proofs on-chain make censorship attempts detectable."
  o: "Reference implementations and circuit source are open source; verification contracts are deployed on public networks. Node participation criteria vary by protocol."
  p: "Inputs remain encrypted unless a threshold of nodes colludes to decrypt. PVSS ensures share publication is verifiable, and on-chain decryption proofs make any premature decryption publicly detectable."
  s: "Security relies on the k-of-n honest-committee assumption, correct implementation of the DKG and PVSS protocols, and soundness of the underlying ZK circuits. A compromised threshold of nodes can decrypt; economic slashing and governance backstops are the residual mitigations."

post_quantum:
  risk: low
  vector: "The PVSS scheme is lattice-based and quantum-safe. The proving layer (Noir/Barretenberg/UltraHonk) uses pairing-based cryptography and would need migration to a quantum-safe proving system; this is expected well before any quantum threat materializes."
  mitigation: "Migrate the proving layer to a quantum-safe proof system when available. Encrypted state itself is not vulnerable."

standards: []

related_patterns:
  composes_with: [pattern-threshold-encrypted-mempool, pattern-pretrade-privacy-encryption]
  alternative_to: [pattern-tee-key-manager]
  see_also: [pattern-ephemeral-committees]

open_source_implementations:
  - url: https://github.com/theinterfold/interfold
    description: "The Interfold protocol: threshold BFV, PV-TBFV, DKG and decryption aggregator circuits"
    language: "Rust, Noir, Solidity"
  - url: https://github.com/shutter-network/shutter
    description: "Shutter threshold-encryption toolkit with on-chain keyper verification"
    language: "Rust, Go"
---

## Intent

Distribute decryption authority across a committee of independent nodes such that no single party can unilaterally decrypt encrypted state, and every cryptographic operation — key generation, share publication, and decryption — produces an on-chain verifiable proof. Misbehavior (invalid shares, premature decryption, refusal to decrypt) is cryptographically detectable and provable for slashing or dispute resolution.

This pattern generalises the distributed key generation and threshold decryption layer used in threshold-encrypted mempools and encrypted execution environments. It differs from basic threshold encryption by making every step publicly verifiable via zero-knowledge proofs, eliminating the need to trust the committee's honesty — you can prove when they misbehave.

## Components

- **Publicly Verifiable Secret Sharing (PVSS)**: Each committee member publishes an encrypted key share along with a ZK proof that the share is correctly formed, so any observer can verify share validity without trusting the publisher.
- **Distributed Key Generation (DKG) protocol**: Committee members jointly produce a shared public key through rounds of PVSS exchanges. No party ever holds the full private key.
- **On-chain DKG verifier**: A smart contract or verifier contract that validates the aggregated DKG proof (e.g., a Noir circuit verifying that the published public key was correctly derived from valid PVSS shares).
- **Threshold decryption aggregator**: Collects decryption shares from committee members (each with a per-share ZK proof of correctness), aggregates them into the plaintext result once the threshold is met, and submits the aggregated proof on-chain.
- **On-chain decryption verifier**: Validates the aggregated decryption proof, confirming that a threshold of valid shares was used to produce the claimed plaintext.
- **Slashing / dispute mechanism**: Economic penalties triggered by on-chain proof of misbehavior (invalid shares during DKG, invalid decryption shares, provable premature decryption).

## Protocol

1. [operator] Committee members are selected (via sortition, governance, or explicit delegation).
2. [operator] Each member generates a PVSS share and publishes it with a ZK proof of correctness. Shares are aggregated into the committee public key.
3. [contract] The DKG verifier validates the aggregated proof on-chain and records the published public key.
4. [user] Data providers encrypt inputs to the committee public key and submit ciphertexts with proofs of valid encryption.
5. [operator] Computation runs over encrypted inputs; a ciphertext output is produced and published.
6. [operator] Each committee member produces a decryption share with a per-share ZK proof; shares are aggregated once the threshold is met.
7. [contract] The decryption verifier validates the aggregated proof on-chain and publishes the plaintext output. Any invalid or missing shares are cryptographically identifiable for slashing.

## Guarantees & threat model

Guarantees:

- **No single point of decryption**: fewer than k committee members cannot decrypt, even if the rest collude.
- **Public verifiability**: every share (DKG and decryption) carries a ZK proof; invalid shares are detectable by any observer without trusting the publisher.
- **Provable misbehavior**: premature decryption, invalid shares, or refusal to decrypt leave on-chain evidence suitable for slashing or dispute resolution.
- **Auditability**: the full chain of cryptographic operations — from DKG to decryption — is anchored on-chain and independently verifiable.

Threat model:

- **Honest-threshold assumption**: if k or more members collude, they can decrypt. Economic penalties and diverse operator selection raise the cost of collusion but do not remove the assumption.
- **Liveness**: if fewer than k members are online, decryption stalls. Timeouts with refund or failure fallbacks are required.
- **ZK circuit soundness**: a bug in the DKG or decryption verifier circuits could allow invalid shares to pass verification. Formal verification and audits are the primary mitigations.
- **Metadata leakage**: transaction volume, timing, and committee membership are observable on-chain regardless of encryption.

## Trade-offs

- **On-chain verification cost**: every DKG and decryption round incurs gas for proof verification. Circuit optimisation and proof aggregation (e.g., folding schemes) reduce but do not eliminate this cost.
- **Coordination latency**: DKG requires multiple rounds of PVSS exchange across committee members, adding latency before the input window can open.
- **Committee management overhead**: operator registration, sortition, bond management, and exit queues add protocol complexity beyond the cryptographic layer.
- **Failure modes**: DKG timeout, insufficient committee members, invalid shares detected mid-protocol — all require fallback states and refund logic in the coordinating contract.
- **Post-quantum**: the encryption layer (PVSS) is lattice-based and quantum-safe. Only the proving layer (Noir/Barretenberg/UltraHonk) relies on pairing-based cryptography and would need migration to a quantum-safe proof system; this is expected well before any quantum threat materializes.

## Example

A sealed-bid auction protocol requests a committee of 5 nodes with a threshold of 3 for decryption. The nodes run PVSS-based DKG and publish their shares on-chain; the DKG verifier contract confirms correctness. Bidders encrypt their bids to the published public key and submit them during the input window. After the auction logic runs over encrypted bids (via FHE or zkVM), each node produces a decryption share of the output with a per-share ZK proof. Once 3 valid shares are aggregated, the decryption verifier confirms the result and publishes the winning price on-chain. A node that submitted an invalid share during DKG or decryption is provably at fault and slashed.

## See also

- [The Interfold Documentation — Cryptography](https://docs.theinterfold.com/cryptography)
- [Shutter Network — Threshold Encryption](https://docs.shutter.network/)
- [PVSS: Publicly Verifiable Secret Sharing (Stadler, 1996)](https://link.springer.com/chapter/10.1007/3-540-68697-5_15)
