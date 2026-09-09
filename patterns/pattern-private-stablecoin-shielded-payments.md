---
title: "Pattern: Private Stablecoin Shielded Payments"
status: ready
maturity: testnet
type: standard
layer: L2
last_reviewed: 2026-06-17

works-best-when:
  - The cash leg must be private in both amounts and counterparties, with selective disclosure available to auditors.
  - Ethereum-native tooling on an L2 or app-chain is preferred, with composability into PvP or DvP flows.
avoid-when:
  - Public transparency of the cash leg is required by policy.
  - Off-chain netting already satisfies the confidentiality requirement.

context: both
context_differentiation:
  i2i: "Between institutions the issuer and both counterparties are KYC-aligned with legal recourse. Permissioned shielded pools and ERC-3643 gating preserve unlinkability within the consortium while viewing keys cover audit. Issuer-level controls (freeze, blacklist) are negotiated contractually and bounded by governance."
  i2u: "For end users the cash leg must not depend on an issuer who can freeze balances unilaterally or on an operator who can block withdrawals. Forced withdrawal and user-controlled viewing keys are required to make the privacy guarantee meaningful; without them, censorship resistance collapses to `low`."

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Reaches `medium` when the privacy L2 has a permissionless exit path and no single issuer can freeze balances without governance. Drops to `low` when issuer controls (freeze, blacklist) are unconstrained or when the L2 has operator-controlled exit."
  o: "Shielded-pool contracts and zero-knowledge circuits are typically open source. Fully Homomorphic Encryption coprocessors and hosted issuer tooling often include proprietary components."
  p: "Amounts, counterparties, and memos are hidden from non-participants. Viewing keys and selective disclosure proofs cover auditors. Network-layer metadata (timing, gas payer, IP) remains visible unless combined with a network-anonymity pattern."
  s: "Rides on the soundness of the underlying proof system or Fully Homomorphic Encryption scheme, the honesty of any threshold decryption committee, and the integrity of the L2 state root anchored to L1."

post_quantum:
  risk: high
  vector: "Pairing-based shielded pools (Groth16, PLONK/KZG) broken by CRQC. HNDL risk applies to encrypted notes and any long-lived audit log encrypted under classical asymmetric keys."
  mitigation: "Migrate to STARK-based shielded pools with hash commitments; re-encrypt long-lived audit logs under post-quantum key encapsulation. See [Post-Quantum Threats](../domains/post-quantum.md)."

visibility:
  counterparty: [amounts, identities, memos]
  chain: [commitments, nullifiers]
  regulator: [full_tx with viewing key]
  public: []

standards: [ERC-20, ERC-3643, ERC-7573, EIP-5564]

related_patterns:
  requires: [pattern-shielding]
  composes_with: [pattern-stealth-addresses, pattern-dvp-erc7573, pattern-private-pvp-stablecoins-erc7573, pattern-erc3643-rwa, pattern-regulatory-disclosure-keys-proofs, pattern-verifiable-attestation, pattern-user-controlled-viewing-keys]
  see_also: [pattern-privacy-l2s, pattern-l2-encrypted-offchain-audit, pattern-forced-withdrawal, pattern-network-anonymity]

open_source_implementations:
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Privacy L2 with private notes and viewing keys for programmable confidential tokens"
    language: Noir
  - url: https://github.com/zama-ai/fhevm
    description: "Confidential ERC-20 reference implementations running on a Fully Homomorphic Encryption coprocessor"
    language: "Solidity, Rust"
---

## Intent

Provide stakeholder-only stablecoin transfers on an L2 or app-chain, with viewing-key or zero-knowledge-proof-based disclosure to regulators and auditors, and the ability to compose atomically with asset legs (DvP) or other cash legs (PvP). Hide amounts, counterparties, and memos from non-participants while still enforcing any issuer-level or regulatory policy bound to the stablecoin.

## Components

- Confidential cash token: either a native private-mode stablecoin or a wrapped form of a public ERC-20 held inside a shielded pool or under encrypted balances.
- Privacy L2 or app-chain: programmable-privacy zero-knowledge L2 (private notes plus viewing keys) or a Fully Homomorphic Encryption runtime (encrypted balances and amounts).
- Stealth-address layer (ERC-5564) to hide payer and payee identities in the Fully Homomorphic Encryption case, since encrypted balances alone do not anonymise counterparties.
- Issuer controls bound to ERC-3643 or equivalent gating: allow-listing, freeze, blacklist where mandated by policy.
- Atomic-settlement hook via ERC-7573 to compose the cash leg with an asset leg (DvP) or another cash leg (PvP).
- Audit and disclosure layer: viewing keys, selective zero-knowledge disclosure proofs, or threshold-decryption keys for regulators; on-chain attestations (EAS) log access events.
- Wallet or custody module, optionally MPC-based; issuer admin keys where policy requires.
- Optional L1 anchor of the encrypted audit log and cutoff oracles.

## Protocol

1. [operator] Optional gating: KYC issuers or custodians attest allow-listed participants via verifiable attestations and enforce ERC-3643 eligibility at mint and transfer.
2. [operator] Mint a native private-mode stablecoin or wrap a public ERC-20 into confidential form (shielded pool or encrypted-balance token).
3. [user] Construct a shielded transfer: generate an encrypted note or ciphertext so that only the payer, payee, and any permitted auditor can view amount and counterparty.
4. [contract] Verify the transfer (zero-knowledge proof or homomorphic evaluation), update commitments or encrypted balances, and record any nullifiers.
5. [auditor] Obtain a scoped view via the payee's viewing key or a selective zero-knowledge disclosure proof; access is logged via attestations.
6. [contract] Optional atomic leg: compose with ERC-7573 to settle DvP against a tokenised asset leg or PvP against another cash leg.
7. [operator] Optional: post commitment roots and encrypted audit log hashes to L1 and archive the append-only audit trail off-chain.

## Guarantees & threat model

Guarantees:

- Hides amounts, counterparties, and memos from non-participants; reveals only to stakeholders in the transfer and any permitted auditor.
- Atomic DvP (cash against asset) or PvP (cash against cash) when composed with ERC-7573.
- Verifiable audit trail via attestations; issuer controls enforce policy without exposing transfer contents.

Threat model:

- Soundness of the underlying proof system or Fully Homomorphic Encryption scheme.
- Honesty of the L2 sequencer or prover and of any threshold decryption committee.
- Non-compromised relayer and paymaster. A colluding relayer can link user IPs to shielded activity; a colluding paymaster can selectively censor.
- Unconstrained issuer controls (freeze, blacklist without governance) break the guarantee for users; address at the policy layer.
- Network-layer metadata (timing, gas payer, IP) is out of scope and must be covered by a network-anonymity pattern.

## Trade-offs

- Zero-knowledge or Fully Homomorphic Encryption proofs add per-transfer cost and latency compared with a public ERC-20 transfer.
- Developer experience varies: non-EVM privacy L2s require circuit DSLs; Fully Homomorphic Encryption stacks depend on coprocessors and oracles.
- Timing and ordering metadata can persist even when contents are hidden; mitigate with batching or delayed anchors.
- Stealth addresses alone do not provide issuer-level policy or full cash-flow privacy; they complement, rather than replace, a shielded cash token.
- Monotonic on-chain growth: commitments or ciphertexts accumulate indefinitely; prover or coprocessor costs drift up over time.

## Example

- A dealer pays weekend margin in a confidential stablecoin on a privacy L2. The venue receives funds without public leakage. The regulator gets read-only access via a time-bound viewing key. The payment leg participates in an ERC-7573 PvP against another stablecoin or a DvP against tokenised collateral.

## See also

- [Aztec](../vendors/aztec.md)
- [Zama](../vendors/zama.md)
- [Fhenix](../vendors/fhenix.md)
- [Inco](../vendors/inco.md)
- [Canton Network press release on weekend USDC cash leg](https://www.canton.network/canton-network-press-releases/digital-asset-complete-on-chain-us-treasury-financing)
- [Aztec programmable-privacy documentation](https://docs.aztec.network/)
- [Zama confidential ERC-20 overview](https://www.zama.ai/post/confidential-erc-20-tokens-using-homomorphic-encryption)
- [Fhenix encrypted computation overview](https://blog.arbitrum.io/fhenix-private-computation/)
