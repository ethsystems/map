---
title: "Pattern: Shielding"
status: ready
maturity: production
type: standard
layer: hybrid
last_reviewed: 2026-09-23

works-best-when:
  - You need confidential transfer amounts and counterparties.
  - Selective disclosure or auditing is required.
avoid-when:
  - Transparency is mandated by design.

context: both
context_differentiation:
  i2i: "Between institutions, both counterparties have legal recourse and KYC-aligned identities. A shielded pool can be permissioned to institutional participants once the anonymity set is large enough to preserve unlinkability within the set. Shielding hides positions from competitors while viewing keys cover audit."
  i2u: "User is asymmetric to the operator. The shielded pool's CR must not depend on an operator who can block withdrawals. Forced withdrawal is required to make the privacy guarantee meaningful; without it, CR is effectively `low`."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Reaches `high` on L1 permissionless shielded pools where users can deposit and withdraw without a gatekeeper. Drops to `low` on permissioned L2s with operator-controlled exit. Relies on relayer and paymaster for transaction submission, both of which can censor."
  o: "Open-source contracts are available; users can fork and run their own relayer. Some L2 deployments are closed or operator-controlled."
  p: "Plausible deniability on amounts, sender, and receiver. Metadata (contract-level patterns, IP addresses, gas payer, timing) remains visible and must be covered at the network layer."
  s: "Rides on the soundness of the zero-knowledge proof system and on-chain state integrity. Operator honesty matters on permissioned deployments."

post_quantum:
  risk: high
  vector: "EC-based commitments (Groth16, PLONK/KZG) broken by CRQC; HNDL risk high for encrypted notes."
  mitigation: "STARK-based shielded pools with hash commitments. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-20, ERC-5564]

related_patterns:
  composes_with: [pattern-stealth-addresses, pattern-erc3643-rwa, pattern-regulatory-disclosure-keys-proofs, pattern-private-information-retrieval]
  see_also: [pattern-forced-withdrawal, pattern-user-controlled-viewing-keys, pattern-network-anonymity]

open_source_implementations:
  - url: https://github.com/Railgun-Privacy/contract
    description: "Railgun shielded pool (L1, production)"
    language: "Solidity, Circom"
  - url: https://github.com/ethsystems/pocs/tree/master/pocs/private-payment
    description: "EthSystems PoC (UltraHonk/Noir, research)"
    language: "Noir, Solidity"
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec Network, privacy L2 with native shielding (production)"
    language: "Noir"
---

## Intent

Enable confidential transfers by shielding balances, sender, and receiver, while still allowing regulators and auditors to verify via selective disclosure (viewing keys, proofs). The logic generalizes beyond ERC-20 to any transferable asset.

## Components

- Shielded pool contract stores commitments, the nullifier set, and a ZK verifier. Deployable on L1 or a shielded L2.
- Hash-based commitments (e.g. Poseidon) hide note contents; notes represent spendable state.
- Nullifier set prevents double-spends by tracking spent notes.
- Prover and verifier, with circuit logic authored in a DSL (Noir, Circom, Halo2, Gnark, etc.); the prover runs client-side.
- Wallet/KMS manages shielded keys and optional viewing keys.
- Relayer/paymaster (optional) for gas abstraction.
- Stealth addresses (optional, ERC-5564) for recipient unlinkability.

## Protocol

1. [user] Deposit assets into the shielded pool contract.
2. [contract] Store the commitment submitted by the user in the commitment set.
3. [user] Construct a shielded transfer: generate a zero-knowledge proof that (a) nullifiers for spent notes are not already in the nullifier set, (b) commitments for spent notes are members of the commitment set, (c) the sum of output note amounts does not exceed the sum of input note amounts, and (d) the sender owns the spending keys.
4. [relayer] Submit the shielded transaction for gas abstraction (optional).
5. [contract] Verify the proof, add new commitments, and add spent nullifiers.
6. [auditor] Verify a specific transfer via the recipient's viewing key.
7. [user] Withdraw by proving ownership of notes and burning them.

## Guarantees & threat model

Guarantees:

- Hides amounts, sender, and receiver from non-participants within the anonymity set.
- Enables scoped disclosures to auditors via viewing keys.
- Anchors state on the underlying chain (L1 or L2) for integrity.

Threat model:

- Soundness of the zero-knowledge proof system.
- Non-censoring sequencer or validator set on shielded L2s.
- Non-compromised relayer and paymaster. The relayer sees raw transaction data (nullifier, commitment, proof) and can link user IPs to shielded activity even though the contents stay hidden. A colluding paymaster can selectively censor.
- Network-layer metadata (timing, gas payer, IP) is out of scope for the shielding layer; combine with network-level anonymity to cover it.

## Trade-offs

- Cost and latency of zero-knowledge proofs (prover time, calldata/blobspace).
- Metadata leakage at the network layer.
- Gas abstraction is mandatory in practice: paying gas from the user's own EOA links the exit address back to the entry, breaking entry-to-exit unlinkability. A relayer or paymaster is required to break that link.
- Client-side query leakage: fetching a note's Merkle inclusion path (or scanning for incoming notes) from a third-party RPC or indexer reveals to that operator which note is about to be spent or read, undoing on-chain unlinkability for users who depend on shared infrastructure. Mitigable with private information retrieval (see [pattern-private-information-retrieval](pattern-private-information-retrieval.md)); institutions running their own full node and indexer sidestep this without PIR.
- Monotonic state growth: commitments accumulate indefinitely, so pool state bloats and proof costs drift up over time.

## Example

- A user shields stablecoins into the pool, transfers them privately to a recipient, and the regulator later verifies the transfer via the recipient's viewing key.

## See also

- [Railgun](../vendors/railgun.md)
- [Hinkal](../vendors/hinkal.md)
- [Aztec](../vendors/aztec.md)
- [Exploring hardened shielded pools](https://ethsystems.org/blog/exploring-hardened-shielded-pools/)
- [Shielded-pool extension PoC](https://github.com/ethsystems/pocs/tree/master/pocs/private-payment/shielded-pool-extension)
