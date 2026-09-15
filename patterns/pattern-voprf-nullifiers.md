---
title: "Pattern: vOPRF Nullifiers"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-06-17

works-best-when:
  - You need deterministic per-scope nullifiers or pseudonyms for credentials and signals without revealing the underlying identifier.
  - You want to bound the damage from client-key compromise. Without the server key, an attacker cannot reconstruct past nullifiers from a leaked client secret alone.
  - You can operate or rely on a threshold committee with clear governance, uptime, and abuse controls.
avoid-when:
  - A local nullifier (hash of user secret with scope) is sufficient and offline linkage from client compromise is acceptable.
  - An online dependency is not acceptable for latency, availability, or regional deployment reasons.
  - Trust and compliance for the threshold committee or operator set cannot be established.

context: both
context_differentiation:
  i2i: "Between institutions the committee is typically contracted between known parties with SLAs and legal recourse. The online dependency is a liveness concern for high-frequency operations and needs active-active deployment. Governance of the committee key material is bilateral or consortium-based."
  i2u: "For users the committee must be permissionless or operator-diverse so that no coalition can recover the base identifier. Economic bonding with slashing and a public audit log of evaluation requests protect against silent collusion. Without operator diversity the user has no recourse if the committee is coerced."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Reaches `high` when the committee is permissionless and bond-backed with slashing. Drops to `low` when a single operator controls the evaluation pipeline or the rate-limit gate."
  o: "The vOPRF construction itself is specified in open IETF standards; threshold implementations vary in licensing. Production deployments may bundle proprietary orchestration and rate-limit logic."
  p: "Input privacy holds against the committee in the vOPRF model. Metadata about when and how often a user requests an evaluation remains visible to the committee and can leak usage patterns."
  s: "Security rides on the OPRF scheme soundness, the threshold key-generation ceremony, and the honest-threshold assumption (fewer than t colluding nodes)."

post_quantum:
  risk: high
  vector: "Current OPRF constructions rely on the DDH assumption in a prime-order elliptic-curve group, which is broken by a CRQC. HNDL risk applies to any recorded blinded request, since a future adversary with quantum capability plus the committee key could de-blind and reconstruct nullifiers."
  mitigation: "Post-quantum OPRF constructions (lattice-based, VDF-based) are in research. Migration requires re-keying the committee, versioning scopes with a key identifier, and accepting larger message sizes. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [RFC-9497, RFC-9576]

related_patterns:
  composes_with: [pattern-shielding, pattern-private-mtp-auth, pattern-verifiable-attestation]
  alternative_to: [pattern-private-set-intersection-oprf]
  see_also: [pattern-zk-promises, pattern-compliance-monitoring]

open_source_implementations:
  - url: https://core.taceo.io/articles/taceo-oprf/
    description: "TACEO threshold vOPRF writeup with reference implementation (research)"
    language: "Rust"
---

## Intent

Generate deterministic, scope-bound nullifiers using a verifiable oblivious pseudorandom function (vOPRF) whose secret key is held by a threshold committee. The client blinds its input, the committee jointly evaluates the OPRF, and the client unblinds the response to derive a nullifier. The server key prevents offline reconstruction from a leaked client secret; verifiability lets the client detect a malicious committee response.

## Components

- Client-side input framing combines a stable identifier (credential identifier, membership secret, device key), a scope (service identifier, action identifier, epoch), and a domain-separation tag into the OPRF input.
- Blinding and unblinding routines implement the hash-to-curve and blind or unblind operations of the chosen OPRF instantiation.
- Threshold vOPRF committee holds the server private key in t-of-n shares. Each node evaluates its share on the blinded input; the client combines t responses into a single evaluation.
- Committee public key used by the client to verify that the combined evaluation corresponds to the advertised key.
- Rate-limit and abuse gate in front of the committee (authenticated tokens, fees, proof-of-eligibility) to prevent exhaustive-query attacks that would reveal mappings.
- Nullifier registry records used nullifiers to enforce one-use-per-scope semantics (on-chain contract or off-chain log).

## Protocol

1. [user] Derive the scoped input x by hashing a domain tag with the base identifier, the scope descriptor, epoch, and chain context.
2. [user] Hash x to a curve point, blind it with a random scalar, and submit the blinded point and scope metadata to the committee along with any required rate-limit token.
3. [operator] Each committee node evaluates its key share on the blinded point and returns its share plus a verifiability proof.
4. [user] Combine t shares, verify the response against the committee public key, and discard the result if verification fails.
5. [user] Unblind the response to obtain y, then derive a domain-separated seed and the final nullifier by hashing.
6. [contract] Register the nullifier in the on-chain registry, or attach it to a zero-knowledge proof or credential presentation as the anti-replay handle.

## Guarantees & threat model

Guarantees:

- Deterministic per-scope output: the same base identifier under the same scope always produces the same nullifier.
- Input privacy against the committee: the committee does not learn the base identifier in the vOPRF model.
- Verifiability: the client can detect a committee response that does not correspond to the advertised public key.
- Reduced offline linkage: a leaked client secret alone cannot reconstruct past nullifiers without live committee access.

Threat model:

- Honest-threshold assumption on the committee. A coalition of t or more nodes can evaluate arbitrary inputs offline and reconstruct all nullifiers for any recorded base identifier.
- An attacker who compromises the client can reissue evaluation requests with the same inputs and regenerate nullifiers. Determinism is a feature, and this path must be mitigated with scoped epochs, access controls, or proof-of-eligibility.
- Abuse and denial of service on the committee. The evaluation endpoint must be gated to prevent committee exhaustion or mapping attacks that brute-force small identifier spaces.
- Committee metadata (timing, IP, volume) is out of scope for the vOPRF layer and must be handled by a network-anonymity layer.

## Trade-offs

- Online dependency: evaluation requires a live committee, adding latency and an availability risk.
- Committee governance overhead: threshold key generation, share rotation, incident response, and key versioning are operational burdens. Rotating keys changes outputs unless scopes include an explicit key identifier.
- Scope design is delicate. Too-broad scopes create cross-service linkability; too-narrow scopes defeat the anti-replay or rate-limit purpose.
- Denial-of-service surface on the committee requires rate-limit tokens, fees, or proof-of-eligibility.

## Example

A KYC issuer gives a user a credential with an internal credential identifier. A downstream service wants to enforce one active session per user per day without learning who the user is. The scope is the hash of the service identifier, the action "daily-session", and the day bucket. The user derives the OPRF input from the credential identifier and the scope, blinds it, and submits to the committee. After verification and unblinding, the user derives a nullifier, and the service records it to reject any duplicate submission within the day.

## See also

- [RFC 9497 (OPRF and VOPRF)](https://www.rfc-editor.org/rfc/rfc9497.html)
- [RFC 9576 (Privacy Pass Architecture)](https://www.rfc-editor.org/rfc/rfc9576.html)
- [TACEO vOPRF writeup](https://core.taceo.io/articles/taceo-oprf/)
- [TACEO Merces vendor page](../vendors/taceo-merces.md)
- [Beasley (Li, Wang, Zhang, ePrint 2026/2010)](https://eprint.iacr.org/2026/2010): an implemented round-optimal, maliciously secure lattice VOPRF over MLWE/MSIS, and a candidate post-quantum replacement for the DDH construction here, benchmarked at 520 ms for the client request and proof, 15.3 ms verification, and 109 KB client communication on one AVX2 core. Bandwidth is orders of magnitude above the elliptic-curve version, and the paper covers a single server rather than the threshold committee this pattern assumes.
