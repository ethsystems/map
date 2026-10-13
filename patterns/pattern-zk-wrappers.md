---
title: "Pattern: ZK Wrappers"
status: ready
maturity: testnet
type: standard
layer: offchain
last_reviewed: 2026-06-18

works-best-when:
  - A signed credential already exists in a PKI or web2 system
  - The signature scheme and issuer public keys are stable and publicly retrievable
  - Holder wants to prove attributes without revealing credential content or identity
avoid-when:
  - The credential source has an API and a TLS or API proof is sufficient
  - Attributes change frequently and no on-chain freshness anchor exists
  - Signature primitive has no practical in-circuit implementation

context: both
context_differentiation:
  i2i: "Between institutions the issuer is typically a regulated party (passport authority, KYC provider, registrar) whose public keys sit in established PKI. Both counterparties can independently verify issuer trust roots, and re-issuance or revocation is handled inside known operational frameworks."
  i2u: "For end users the issuer is often a third party the user does not control (passport agency, DKIM-signing email provider). Users depend on the issuer's continued availability and on key-rotation discipline; if the issuer revokes or rotates keys, users may need to re-prove with no recourse and no warning."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Holders depend on the issuer for new issuance and key rotation. Existing signed credentials stay provable after issuer failure, but revocation and refresh depend on the issuer unless paired with an on-chain freshness anchor."
  o: "In-circuit verifiers and proving backends (Noir/Barretenberg, Circom/Groth16, Halo2, SP1) are open source; some issuer tooling and key sources are not."
  p: "Only the declared predicates over the signed payload are revealed; credential content, signature bytes, and the holder's direct link to the issuer stay hidden."
  s: "Rides on the soundness of the proof system (and any trusted setup) and the security of the wrapped signature scheme; a wrapped signature is only as strong as its underlying primitive."

post_quantum:
  risk: high
  vector: "Two independent exposures: the wrapped signature (RSA, ECDSA) is broken by a CRQC (a wrapped signature is only as strong as its underlying scheme), and the proving backend itself, when pairing/EC-based (Groth16, UltraHonk over BN254, KZG), is also broken by a CRQC. Transparent hash-based systems are not affected."
  mitigation: "Migrate the underlying credential signatures to post-quantum schemes and use a hash-based or STARK proof system; pair with an on-chain freshness anchor for re-issuance. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-verifiable-attestation, pattern-zk-kyc-ml-id-erc734-735]
  see_also: [pattern-zk-tls, pattern-voprf-nullifiers, pattern-regulatory-disclosure-keys-proofs]

open_source_implementations:
  - url: https://zkpassport.id/
    description: "ZKPassport: passport SOD signature proofs"
    language: "Noir, UltraHonk"
  - url: https://self.xyz/
    description: "Self: identity credential proofs"
  - url: https://github.com/anon-aadhaar
    description: "Anon Aadhaar: UIDAI signature proofs"
    language: "Circom, Groth16"
  - url: https://prove.email/
    description: "zkEmail: DKIM signature proofs"
    language: "Circom, Noir, SP1"
  - url: https://github.com/zkproofport/proofport-app
    description: "ZKProofport: mobile/on-device ZK predicates over signed attestation credentials using Noir, UltraHonk, and mopro for iOS/Android proving, with EVM verification"
    language: "TypeScript, Rust, Noir, Solidity"
---

## Intent

Verify an off-chain digital signature over an existing credential inside a ZK circuit. The verifier learns that a valid signature exists and that chosen predicates hold over the signed payload, and nothing else.

## Components

- Signature verification in-circuit: RSA-PKCS1v1.5 and RSA-PSS (up to 4096-bit), ECDSA over P-256 / P-521 / Brainpool curves, EdDSA.
- Hash primitive: SHA-256, SHA-384, SHA-512 depending on credential format.
- Issuer public key source: passport CSCA master list, UIDAI RSA key, DKIM DNS records, EAS issuer address, W3C VC issuer DID.
- Proof system: Noir with Barretenberg (UltraHonk over BN254), Circom with Groth16, Halo2, SP1 zkVM.
- Public inputs: issuer key identifier, predicate result, optional scope-bound nullifier.

## Protocol

1. [user] Obtain the signed credential. Examples: NFC read of a passport's SOD, DKIM-signed email in a mailbox, attestation fetched from EAS.
2. [user] Load the credential, the signature, and the issuer public key into the prover. The key is either a public input or compiled into the circuit.
3. [prover] The circuit verifies the signature over the payload, parses the relevant fields, and evaluates predicates such as nationality equals X, domain equals Y, age is at least N.
4. [prover] Output a proof with public inputs for the predicate result, issuer key identifier, and optional nullifier binding the proof to a verification context.
5. [contract] The verifier contract or service checks the proof and accepts the predicate. No credential bytes cross the wire.

## Guarantees & threat model

Guarantees:

- Hides credential content, signature bytes, the full signed payload, and the holder's direct link to the issuer.
- Proves a valid signature from a specific issuer key over a payload satisfying the declared predicates.
- Selective disclosure: predicates reveal only the chosen attributes (nationality, age threshold, domain) while the remaining credential fields stay hidden.

Threat model:

- Issuer honesty and key custody: a compromised issuer key can sign false credentials that wrap into valid proofs.
- Soundness of the proof system and correctness of the in-circuit signature verification and field parsing.
- Issuer availability for revocation and re-issuance: without an on-chain freshness anchor, a stale credential can remain provable after the underlying fact changes.

## Trade-offs

- Circuits for non-arithmetic-friendly primitives are large. RSA-4096 with SHA-512, or ECDSA over P-521 emulated in BN254, reach millions of constraints. Proof generation on mobile is feasible but not cheap.
- The issuer still controls new issuance. Existing signed credentials remain provable after issuer failure, but revocation and refresh depend on the issuer unless paired with an on-chain anchor. See [Resilient Identity Continuity](../use-cases/resilient-identity-continuity.md).
- Trusted setup depends on the proof system. Groth16 is per-circuit with ceremony requirements; UltraHonk needs a one-time universal (updatable) SRS rather than a per-circuit ceremony; Halo2 is transparent with IPA.
- Issuer key rotation is a liveness risk. Circuits or on-chain registries need to accept the new key without invalidating historical proofs.

## Example

An investor proves OFAC non-sanctioned status to an issuance contract using a [ZKPassport](https://zkpassport.id/) proof over the passport SOD signature, RSA-4096 with SHA-512 or ECDSA over a Brainpool curve depending on the issuing state. The contract receives public inputs `{country is X, sanctions list non-member, issuer key fingerprint}` and accepts. No passport data, name, or date of birth is revealed.

## See also

- [OpenAC (eprint 2026/251)](https://eprint.iacr.org/2026/251): transparent, no trusted setup
- [zk-creds (Ian Miers et al., eprint 2022/878)](https://eprint.iacr.org/2022/878)
