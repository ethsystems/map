---
title: "Pattern: Ephemeral Committees & Disposable Encrypted State"
status: draft
maturity: testnet
type: standard
layer: offchain
last_reviewed: 2026-07-21

works-best-when:
  - Encrypted state must become permanently inaccessible after a defined window, without relying on storage deletion or operational hygiene.
  - Each computation should be isolated from others — key material from one execution must not compromise another.
  - The data disposal guarantee must combine cryptographic and procedural mechanisms — key destruction renders ciphertexts undecryptable, but honest disposal by node operators is required since no cryptographic protocol can force data deletion.
avoid-when:
  - Persistent encrypted state is needed across multiple computations or time periods — use long-lived key management instead.
  - The committee cannot be trusted to discard keys (the guarantee depends on honest disposal).
  - Recovery or rollback of encrypted state after the committee window closes is a hard requirement.

context: both
context_differentiation:
  i2i: "Between institutions, disposable state means no long-term key custody liability for either party. After the committee disbands, neither side can be compelled to decrypt historical data because the key material no longer exists anywhere."
  i2u: "For end users, the guarantee that their encrypted inputs will become permanently inaccessible after the computation protects against future data breaches or compelled disclosure. The guarantee is only as strong as the honest-majority key disposal assumption."

crops_profile:
  cr: high
  o: yes
  p: full
  s: high

crops_context:
  cr: "No single operator can prevent key disposal or retain key material — but the CROPS guarantee relies on honest-majority disposal. If a majority retains keys, undecrypted state remains potentially accessible."
  o: "Protocol logic and lifecycle enforcement are open source and verifiable on-chain. Key disposal is an off-chain action by operators and cannot be cryptographically proven — it relies on economic incentives and reputation."
  p: "Privacy reaches `full` after the committee disbands: encrypted state becomes information-theoretically inaccessible if keys are truly discarded. During the active window, privacy is `partial` — a threshold of nodes could collude to decrypt."
  s: "Security during the active window depends on the k-of-n threshold. After disposal, security depends on honest key destruction — an assumption that cannot be cryptographically verified, only economically incentivized."

post_quantum:
  risk: medium
  vector: "Encrypted state collected during the active window has HNDL exposure if the underlying encryption is not post-quantum. State after key disposal is information-theoretically lost regardless of quantum capability, assuming honest disposal."
  mitigation: "Combine with post-quantum encryption schemes. The disposal guarantee itself is not quantum-dependent."

standards: []

related_patterns:
  requires: [pattern-verifiable-dkg-threshold-decryption]
  composes_with: [pattern-pretrade-privacy-encryption]
  alternative_to: [pattern-tee-key-manager]
  see_also: [pattern-forward-secure-signatures]

open_source_implementations:
  - url: https://github.com/theinterfold/interfold
    description: "The Interfold protocol: E3 lifecycle with ephemeral committee key generation and mandatory disposal"
    language: "Rust, Noir, Solidity"
---

## Intent

Bound the accessibility of encrypted state to the lifetime of a specific committee, then render all undecrypted state permanently inaccessible by having honest committee members discard their key material as toxic waste after their duties complete. This provides cryptographic data disposal without relying on storage deletion, operational hygiene, or trusted custodians — once the keys are gone, the state is gone.

This pattern is the foundation for systems where encrypted inputs must leave no decryptable trace after the computation finishes: sealed-bid auctions where losing bids should become permanently unreadable, secret ballots where individual votes must be destroyed after tallying, or cross-institutional data analysis where raw inputs must not persist in decryptable form.

## Components

- **Ephemeral committee**: A group of nodes selected for a single computation (one E3, one auction, one ballot). The committee exists only for the duration of that computation.
- **Single-use key material**: The DKG produces a public key used exclusively for one computation. After decryption, the corresponding private key shares serve no further purpose.
- **Key disposal (toxic waste) protocol**: After the output is decrypted and published, honest committee members securely delete their key shares. The act is off-chain and cannot be cryptographically proven, so it is reinforced by economic incentives (slashing for any provable future use of the key material).
- **Timeout-enforced lifecycle**: The coordinating contract enforces strict deadlines (committee formation, DKG, input window, compute, decryption). If any phase times out, the E3 fails and keys should still be discarded. If compute never finishes, encrypted inputs become permanently inaccessible.
- **Slashing for intermediate decryption**: Attempting to decrypt anything other than the agreed-upon output (inputs, intermediate state, or outputs from other computations) is slashable. Combined with key disposal, this creates both ex-ante (economic) and ex-post (key destruction) barriers to unauthorized decryption.

## Protocol

1. [operator] A committee is formed for a specific computation request (via sortition or explicit selection).
2. [operator] The committee runs DKG to produce a shared public key. Each member holds a private key share valid only for this committee.
3. [user] Data providers encrypt inputs to the committee public key during the input window. No other committee's key can decrypt these inputs.
4. [operator] The compute provider executes the program over encrypted inputs and publishes the ciphertext output.
5. [operator] The committee performs threshold decryption of the output and publishes the plaintext result.
6. [operator] Honest committee members securely delete (overwrite, shred) their private key shares. These shares are now toxic waste — retaining them is a liability, not an asset.
7. [contract] The E3 lifecycle completes. Any encrypted state that was not decrypted during the active window (inputs from non-winning bidders, intermediate ciphertexts, abandoned computations) is now permanently inaccessible — the keys to decrypt it no longer exist.

## Guarantees & threat model

Guarantees:

- **Time-bounded decryption window**: encrypted state is only accessible while a threshold of committee members possess their key shares. After the window closes and keys are discarded, decryption becomes impossible under the honest-majority assumption.
- **Cryptographic data disposal**: undecrypted state becomes permanently inaccessible without requiring storage deletion. Even if ciphertexts are retained indefinitely, they cannot be decrypted once the keys are gone.
- **Isolation across computations**: each committee uses fresh key material, so compromise of one committee's keys does not expose state from any other computation.
- **Economic backstop**: slashing conditions for intermediate decryption and key retention create a cost for misbehavior that complements the cryptographic disposal guarantee.

Threat model:

- **Key retention**: if a threshold of committee members retains their key shares instead of discarding them, undecrypted state remains accessible. This cannot be cryptographically prevented — it relies on honest behavior, economic incentives, and the fact that retained keys are a liability (provable use triggers slashing).
- **Key exfiltration before disposal**: if an attacker compromises a threshold of nodes during the active window and exfiltrates their key shares before disposal, the state remains accessible to the attacker regardless of subsequent disposal.
- **Liveness during active window**: if the committee fails before decryption (timeout, insufficient online members), encrypted state may become permanently inaccessible — which is the desired guarantee for privacy, but a failure mode for the computation's utility.
- **Disposal verification gap**: unlike DKG and decryption (which produce on-chain verifiable proofs), key disposal is an off-chain action with no cryptographic proof. The guarantee rests on incentives, not verification.

## Trade-offs

- **No recovery**: once the committee disbands and keys are discarded, undecrypted state is irrecoverable. This is the intended guarantee, but it means there is no "undo" for failed computations or accidental early disposal.
- **Honest-majority disposal assumption**: the cryptographic disposal guarantee is only as strong as the honest-majority key destruction commitment. Retained key shares (maliciously or accidentally) undermine the guarantee.
- **Fresh DKG per computation**: generating new key material for every computation adds latency and on-chain verification cost compared to long-lived keys.
- **Liveness risk**: if the committee fails to reach threshold during decryption, the output is lost. Timeout mechanisms and refund logic in the coordinating contract mitigate economic harm but cannot recover the lost output.
- **Key destruction hygiene**: secure deletion in practice (memory overwrite, hardware security module zeroization, filesystem shredding) is operationally non-trivial and varies across deployment environments.

## Example

A sealed-bid auction uses an ephemeral committee of 5 nodes with threshold 3 for decryption. The committee generates a fresh public key used only for this auction. Bidders encrypt their bids; the compute provider determines the winner over encrypted bids and publishes the ciphertext result; the committee decrypts the winning price and publishes it. After decryption, all 5 nodes securely delete their key shares. The losing bids — still encrypted on-chain — are now permanently inaccessible: no key material exists anywhere that can decrypt them. Even if the auction contract's storage is preserved for years, the losing bids can never be revealed.

## See also

- [The Interfold — E3 Lifecycle](https://docs.theinterfold.com/computation-flow)
- [The Interfold — Exits, Rewards & Slashing](https://docs.theinterfold.com/ciphernode-operators/exits-and-slashing)
- [Forward Secure Signatures pattern](../patterns/pattern-forward-secure-signatures.md) — related concept: keys that lose utility over time
