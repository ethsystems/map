---
title: "Pattern: Private Client Authentication for Institutional EOAs"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-09-30

works-best-when:
  - Institutions must comply with KYC/AML but want to protect client privacy on-chain.
  - Clients need to prove control of EOAs without revealing identity.
  - Multi-address support is desired (same seed, different EOAs).
avoid-when:
  - Regulators require explicit PII disclosure on-chain.
  - Institutions lack capacity to manage zero-knowledge infrastructure or Merkle registry.

context: both
context_differentiation:
  i2i: "Between institutions, peers can negotiate mutual inclusion terms and publish roots to shared registries. Authentication is typically bilateral, with each institution trusting the other's KYC process, and disputes have legal recourse."
  i2u: "For end users, the institution is the sole gatekeeper of client inclusion. Users have no unilateral escape path if removed from the registry. Censorship resistance depends on how the Merkle root is governed (single issuer, multi-institution registry, or DAO)."

crops_profile:
  cr: low
  o: yes
  p: full
  s: high

crops_context:
  cr: "Reaches `medium` or `high` when Merkle membership is managed by a decentralized registry or multi-institutional DAO rather than a single institution. Drops to `low` when a single institution controls inclusion and root updates with no user recourse."
  o: "Zero-knowledge circuits and attestation schemas are open source. Institutional registries and KYC backends are typically proprietary."
  p: "Hides client identity, KYC details, and linkage between EOAs of the same client. On-chain observers see only the proof and the nullifier."
  s: "Rides on the soundness of the zero-knowledge proof system, collision-resistance of the Merkle hash function, and honest tree construction by the institution. Nullifiers prevent replay but do not prevent a malicious issuer from inserting fake leaves."

post_quantum:
  risk: high
  vector: "Pairing-based and elliptic-curve proof systems (Groth16, PLONK/KZG) are broken by a CRQC, letting an attacker forge membership proofs. HNDL risk is bounded because the Merkle tree itself uses hash-based commitments."
  mitigation: "Migrate to STARK-based membership circuits with hash-based commitments. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-3643, ERC-735]

related_patterns:
  composes_with: [pattern-verifiable-attestation, pattern-voprf-nullifiers, pattern-regulatory-disclosure-keys-proofs, pattern-zk-kyc-ml-id-erc734-735]
  see_also: [pattern-zk-tls, pattern-user-controlled-viewing-keys]

open_source_implementations:
  - url: https://github.com/semaphore-protocol/semaphore
    description: "Semaphore: zero-knowledge signaling with Merkle membership and nullifiers"
    language: "Circom, Solidity"
  - url: https://github.com/privacy-scaling-explorations/zk-kit
    description: "PSE zk-kit: Merkle tree libraries used by Semaphore and similar membership-proof systems"
    language: "TypeScript"
---

## Intent

Authenticate client externally-owned accounts in a way that satisfies regulatory KYC/AML requirements while preserving on-chain privacy. The institution maintains a Merkle tree of approved client EOAs; clients prove inclusion with a zero-knowledge proof plus a nullifier. Observers see neither the client's identity nor links between the client's other EOAs.

## Components

- Off-chain KYC process operated by the institution or a regulated distributor, producing a per-client approval record.
- Merkle tree of approved EOAs maintained off-chain and committed on-chain as a root.
- On-chain root registry contract that holds the current and recent roots for proof verification.
- Membership circuit that proves Merkle inclusion, binds a nullifier to the leaf, and proves control of the private key for the EOA.
- Nullifier set stored on-chain or in the verifier contract to prevent proof replay.
- Client wallet with zero-knowledge prover integration for generating membership proofs.
- Audit interface so regulators can verify tree updates and proof validity without learning which leaf belongs to whom.

## Protocol

1. [institution] Run KYC off-chain with the client and record approval.
2. [institution] Insert the client's EOA into the approved-set Merkle tree and publish the updated root on-chain.
3. [user] Generate a membership proof: Merkle path from the client's leaf to the current root, a nullifier bound to the leaf, and a signature proving control of the EOA.
4. [user] Submit the proof and nullifier to the institution's verifier contract or off-chain verifier.
5. [contract] Verify the proof against the published root, check the nullifier has not been used, and record the nullifier.
6. [regulator] Audit the tree, the nullifier set, and sampled proofs without learning the mapping from leaves to clients.

## Guarantees & threat model

Guarantees:

- Hides client identity, KYC record, and linkage across the client's other EOAs.
- Proves Merkle membership in the approved set, control of the corresponding private key, and non-replay via the nullifier.
- Scoped disclosure to regulators: the registry operator can selectively reveal the mapping between leaves and client identities under a legal process.

Threat model:

- Soundness of the zero-knowledge proof system and the Merkle hash function.
- Honest tree construction by the institution. A dishonest issuer can insert unapproved leaves, inflating the anonymity set with accounts that did not go through KYC.
- Non-censoring root publication path. If the institution freezes root updates, newly-approved clients cannot authenticate.
- Out of scope: network-layer metadata (IP, timing), on-chain behavioural fingerprints after authentication, and deanonymisation via transaction graph analysis.

## Trade-offs

- Proof generation adds latency (seconds on commodity hardware, longer in browser WASM). Wallets need a prover integration.
- Tree updates must be frequent enough for newly-approved clients to transact. Batch cadence is a governance parameter.
- Current Semaphore-style membership circuits do not natively support proving that several EOAs derive from the same seed. Multi-address support requires an extended circuit or a separate attestation layer.
- Revocation requires a root update and careful handling of in-flight proofs against the previous root.

## Example

A custodian runs a KYC registry and inserts approved client EOAs into a Merkle tree. A client generates a membership proof showing inclusion in the current root, bound to a fresh nullifier. The custodian's contract validates the proof before processing a tokenised-security transfer. External observers see the transaction and a nullifier but cannot link the EOA to a specific client or discover the client's other addresses. The regulator can later audit the registry root, the nullifier set, and sampled inclusion proofs.

## See also

- [Semaphore protocol](https://semaphore.pse.dev/): Merkle membership, nullifiers, and signaling primitives used by this pattern
- [ERC-3643: T-REX permissioned token standard](https://eips.ethereum.org/EIPS/eip-3643)
- [EAS: Ethereum Attestation Service](https://attest.org/)
- [Approach: Private Identity](../approaches/approach-private-identity.md)
