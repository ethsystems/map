---
title: "Pattern: Verifiable Attestation"
status: ready
maturity: production
type: standard
layer: hybrid
last_reviewed: 2026-09-18

works-best-when:
  - Smart-contract logic must gate on off-chain attested facts (KYC status, accreditation, membership).
  - A user wants to prove a statement about their identity without revealing the underlying data.
  - A trusted issuer exists whose signature the verifier is willing to rely on.
avoid-when:
  - No trusted issuer is available or issuer centralization is unacceptable.
  - Simple token gating or an allowlist would meet the requirement with less complexity.

context: both
context_differentiation:
  i2i: "Between institutions both counterparties can cross-attest or rely on mutual-recognition frameworks. Issuer identity is public and issuers sit within a regulated perimeter, so attestation quality is backed by legal recourse. The CR constraint is mild because peers can route around a single issuer."
  i2u: "A user cannot self-attest and depends on an issuer that may refuse, delay, or revoke service. CR improves only when multiple competing issuers exist and credentials are portable across them (W3C Verifiable Credentials). The consuming contract should treat issuer outage as a user-facing liveness failure."

crops_profile:
  cr: low
  o: yes
  p: partial
  s: high

crops_context:
  cr: "The user depends on the issuer to produce or refresh attestations. CR lifts to `medium` when credentials are portable across competing issuers under a shared schema."
  o: "EAS and W3C Verifiable Credentials are open standards with multiple implementations; ONCHAINID implements ERC-734/735, which remain unmerged draft proposals. Users can run their own verifier and swap issuers."
  p: "On-chain verification reveals that the user holds an attestation of a given type at a given time. Wrapping the attestation in a zero-knowledge proof lets the user disclose only the predicate (over 18, accredited) without revealing the raw claim."
  s: "Rides on issuer key custody, a sound signature scheme (ECDSA, typically over an EIP-712 typed-data digest), and the revocation registry being queryable on-chain."

post_quantum:
  risk: high
  vector: "ECDSA signatures on attestations are broken by a CRQC. An attacker could forge attestations once issuer public keys are exposed. HNDL does not apply to signatures themselves, but long-lived attestations exposed on-chain may need re-issuance before a CRQC arrives."
  mitigation: "Migrate issuer signing to a post-quantum scheme once standardized, for example ML-DSA. Rotate issuer keys on a defined schedule. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [EAS, ERC-734, ERC-735, EIP-712, W3C-VC]

related_patterns:
  composes_with: [pattern-zk-kyc-ml-id-erc734-735, pattern-regulatory-disclosure-keys-proofs, pattern-erc3643-rwa, pattern-zk-wrappers]
  see_also: [pattern-zk-tls, pattern-private-mtp-auth]

open_source_implementations:
  - url: https://github.com/ethereum-attestation-service/eas-contracts
    description: "Ethereum Attestation Service contracts (production)"
    language: "Solidity"
  - url: https://github.com/onchain-id/solidity
    description: "ONCHAINID reference implementation of ERC-734/735 (production)"
    language: "Solidity"
---

## Intent

Let smart contracts verify claims about an identity or off-chain fact without the contract or the public learning the underlying data. Trusted issuers sign attestations off-chain; verifiers check issuer signatures and revocation state on-chain and gate access on the result.

## Components

- Attestation registry stores on-chain records of issued attestations, revocations, and issuer metadata. Deployable on L1 or L2.
- Issuer keys held by a regulated party (bank, KYC provider, regulator) to sign structured attestation payloads.
- Typed signing format binds attestation fields to the issuer signature so that replay and field substitution are infeasible.
- Subject identifier (on-chain address, decentralized identifier, or pseudonymous handle) links the attestation to the party presenting it.
- Revocation list or on-chain state lets issuers invalidate a previously issued attestation.
- Verifier contract checks the issuer signature, the revocation state, and expiry, and emits a gate decision.
- Zero-knowledge wrapper (optional) lets the subject prove a predicate over the attestation without revealing the attestation itself.

## Protocol

1. [issuer] Issue a signed attestation to the subject describing a claim (accredited, KYC cleared, AML screened) with expiry and subject binding.
2. [issuer] Publish the attestation or its hash to the attestation registry, or leave it off-chain with the subject holding the raw credential.
3. [user] Present the attestation (or a zero-knowledge proof of the desired predicate) to a verifier contract when requesting access.
4. [contract] Verify the issuer signature, check the registry for revocation, and enforce expiry.
5. [contract] Grant access, unlock funds, or allow the gated function call on success.
6. [issuer] Revoke the attestation if the underlying fact changes; the next presentation by the subject will fail.

## Guarantees & threat model

Guarantees:

- Minimal disclosure: only the claim needed for the access decision is verified on-chain.
- Non-forgeability: cryptographic signatures bind the claim to the issuer.
- Revocability: issuers can invalidate attestations on-chain; verifiers see the current state.
- Auditability: issuance and access-check events are observable on-chain (or via issuer logs) and are available to regulators.

Threat model:

- Issuer honesty and key custody. A compromised issuer key can produce forged attestations until the key is revoked.
- Availability of the revocation registry at verification time. A stale read defeats revocation.
- Signature-scheme soundness. ECDSA, whether over a raw hash or an EIP-712 typed-data digest, is classically secure and PQ-vulnerable.
- On-chain visibility of the verification event reveals which contract the user interacted with and when. Network-layer privacy and zero-knowledge wrappers are out of scope for this pattern.

## Trade-offs

- Gas cost for signature verification and registry lookup on every access check. Zero-knowledge wrappers move cost from calldata to proof verification.
- Dependence on issuer availability and willingness to issue or refresh attestations.
- Multiple competing standards (EAS, W3C Verifiable Credentials, ONCHAINID) limit portability; adapters or multi-standard verifier contracts are required.
- Verification events leak contract interaction patterns; combine with a privacy L2 or a zero-knowledge wrapper to reduce observer linkability.

## Example

A buyer on a tokenized bond platform holds an accredited-investor attestation issued by a KYC provider and registered on-chain. When the buyer submits a subscription, the bond contract verifies the issuer signature, checks that the attestation is neither revoked nor expired, and clears the transfer. The public sees that an accredited-investor gate was passed, not the buyer's financial details.

## See also

- [EAS (Ethereum Attestation Service)](https://attest.org/)
- [W3C Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/)
- [ERC-734 (draft, unmerged)](https://github.com/ethereum/EIPs/issues/734)
- [ERC-735 (draft, unmerged)](https://github.com/ethereum/EIPs/issues/735)
- [ONCHAINID reference implementation of ERC-734/735](https://github.com/onchain-id/solidity)
- [EIP-712](https://eips.ethereum.org/EIPS/eip-712)
- [Approach: Private Bonds](../approaches/approach-private-bonds.md)
- [Domain: Identity and Compliance](../domains/identity-compliance.md)
