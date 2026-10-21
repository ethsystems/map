---
title: "Pattern: zk-TLS"
status: ready
maturity: testnet
type: standard
layer: offchain
last_reviewed: 2026-10-21

works-best-when:
  - Data required on-chain only lives behind a TLS-protected website with no usable API.
  - The subject can complete an interactive session that produces a TLS transcript (balance, account status, KYC attribute).
  - A single trusted notary (or a multi-party notary) is acceptable within the deployment's threat model.
avoid-when:
  - An authenticated API or a signed credential already exists for the same data; a notarized TLS session adds unnecessary complexity.
  - No notary party is acceptable, or notary availability cannot be guaranteed.

context: both
context_differentiation:
  i2i: "Between institutions the notary is typically the consuming institution itself or a mutually agreed third party under contract. The transcript covers a well-defined data field (balance, identity attribute) whose schema is agreed in advance."
  i2u: "For users the notary sees that a session happened and may see connection metadata. A notary that can refuse service, censor specific origins, or collude with the origin is the main CR threat; multi-notary designs reduce this but do not eliminate it."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "The user depends on a notary that can decline service or collude with the origin website. Lifts to `medium` with a multi-notary construction or a permissionless notary set."
  o: "The core notarized-TLS stack is open source. Production notary infrastructure and the downstream proving tooling vary in licensing."
  p: "The user discloses the notarized transcript fields, not the full session. A zero-knowledge wrapper lets the user prove predicates over the transcript (balance above a threshold, attribute equal to a value) without revealing the raw value to the verifier."
  s: "Rides on TLS session integrity under the chosen notary construction, soundness of the zero-knowledge proof system, and the origin website's own security. A compromised origin produces notarized transcripts that are technically valid but semantically false."

post_quantum:
  risk: high
  vector: "Current notarized-TLS constructions use elliptic-curve Diffie-Hellman for the TLS handshake inside an MPC or 2PC protocol, and notaries sign with ECDSA (secp256k1 or P-256 in TLSNotary). Both the handshake and the MPC channels are broken by a CRQC, so HNDL applies to any recorded session whose redacted contents must stay confidential. Notary signatures face forgery risk, so attestations stop being verifiable evidence once a CRQC exists."
  mitigation: "PQ-safe TLS handshakes require ML-KEM inside the MPC layer, which is an unsolved problem in current notarized-TLS designs. The hybrid ML-KEM key agreement groups are defined only for TLS 1.3, which TLSNotary does not yet support; plan for re-notarization of long-lived data. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-verifiable-attestation, pattern-zk-kyc-ml-id-erc734-735, pattern-erc3643-rwa, pattern-dvp-erc7573]
  see_also: [pattern-private-mtp-auth, pattern-tls-payment-bridge, pattern-zk-proof-systems]

open_source_implementations:
  - url: https://github.com/tlsnotary/tlsn
    description: "TLSNotary Rust implementation: MPC-TLS and proxy mode, TLS 1.2 only; alpha, README says not for production; notary server removed in alpha.13"
    language: "Rust"
---

## Intent

Export verifiable data or identity attributes from a TLS-protected website into a form that an on-chain contract or a counterparty can check. The user jointly runs a TLS session with a notary; the notary signs a transcript of the session; the user produces a zero-knowledge proof over the signed transcript that discloses only the fields required downstream.

## Components

- Origin server hosts the TLS-protected website holding the data of interest (account balance, identity attribute, certification).
- Notary participates in the TLS session alongside the user under an MPC or 2PC protocol, so that neither party alone can forge the transcript. The notary signs commitments to the transcript and the server identity without seeing the plaintext.
- User client drives the session, produces the zero-knowledge proof over the signed transcript, and redacts the fields that should not be disclosed.
- Proof backend (a zero-knowledge proof system) compiles the transcript predicate into a circuit and produces a proof a verifier can check.
- On-chain verifier contract or counterparty verifier checks the notary signature on the transcript and the zero-knowledge proof that the disclosed fields follow from the signed content.

## Protocol

1. [user] Initiate a TLS session to the origin server with the notary participating in the handshake under the MPC or 2PC protocol.
2. [notary] Participate in the session so that the session key is shared; neither party holds the full key, and only the user sees the plaintext.
3. [user] Request the target data from the origin within the session; receive the encrypted response.
4. [notary] Sign an attestation over the user's commitments to the transcript and the server identity, bound to the notary key.
5. [user] Generate a zero-knowledge proof that the disclosed fields (predicate over account balance, identity attribute) follow from the signed transcript, while redacting everything else.
6. [verifier] Check the notary signature and the zero-knowledge proof against the public statement; accept or reject on that basis.

## Guarantees & threat model

Guarantees:

- Authenticity of the disclosed field: the verifier learns that the origin actually served the data under a valid TLS session.
- Minimal disclosure: only the fields covered by the zero-knowledge predicate are visible to the verifier.
- Chain anchoring: the notary signature and the proof can be verified by an on-chain contract.

Threat model:

- Notary honesty under the chosen construction. A corrupted notary colluding with the origin or with the user can produce transcripts that disclose more than the user intended or that falsely certify content.
- Origin honesty for fields the protocol cannot independently verify. If the origin serves false data, the notarized transcript is technically valid but semantically wrong.
- Liveness of the notary and the origin during the session; a dropped MPC round forces a restart.
- Out of scope: the contents of the origin's database, long-term retention of notarized transcripts, and network-layer metadata around the session.

## Trade-offs

- The notary is a trust and liveness dependency. Multi-notary or threshold-notary designs reduce collusion risk at the cost of protocol complexity.
- MPC-TLS session bandwidth and latency are an order of magnitude above a direct TLS session; large transcripts require careful redaction to keep proof cost reasonable.
- Origin-side changes (HTML structure, API response format) break the downstream parser; deployments need a versioning plan for predicates.
- TLSNotary supports only TLS 1.2, with TLS 1.3 on its roadmap; origins that accept only TLS 1.3 are out of reach, and adoption is bounded by that compatibility.

## Example

A custodian needs to verify that an investor holds a minimum balance at a regulated brokerage before allowing a bond subscription. The investor runs a notarized TLS session against the brokerage portal with the custodian acting as notary (or a mutually agreed third-party notary). The investor then produces a zero-knowledge proof that the balance field in the transcript is above the threshold. The custodian verifies the notary signature and the proof, and allows the subscription without ever seeing the exact balance.

## See also

- [TLSNotary documentation](https://tlsnotary.org/)
- [Notarization, TLSNotary docs](https://tlsnotary.org/docs/protocol/notarization)
- [Lightweight Authentication of Web Data via Garble-Then-Prove (Xie et al., USENIX Security 2024)](https://eprint.iacr.org/2023/964)
