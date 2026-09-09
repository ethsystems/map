---
title: "Pattern: Private Intent-Based Vaults"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-06-18

works-best-when:
  - Institutions or funds need private strategy execution while deposited assets remain transparently custodied.
  - Strategies are intent-driven or automated and parameters must not leak to competitors.
  - Auditors can be served via viewing keys or attestation logs without public disclosure of strategy details.
avoid-when:
  - Regulation requires full on-chain transparency of strategy composition and yield sources.
  - The target chain lacks mature privacy infrastructure and low-cost confidential computation.

context: both
context_differentiation:
  i2i: "Between institutions, vault operators and solvers are typically known and contractually bound. LPs can demand audit rights, independent verification of solver code, and attested execution logs. Residual risk (misrouted flow, solver front-running) is bounded by legal recourse."
  i2u: "For users, vault operators are third parties the user cannot audit. Censorship resistance is low because withdrawal typically depends on operator cooperation. Permissionless entry, open-source solver code, a forced-exit path, and multiple competing solvers are required to raise the guarantee above `low`."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Stays `low` as long as entry, exit, or solver selection depend on a single operator. Rises when vault entry is permissionless, multiple independent solvers compete, and a forced-exit path allows redemption without operator cooperation."
  o: "Vault contracts are typically open source. Solver logic and strategy engines may be proprietary; the confidential-computation substrate (Fully Homomorphic Encryption coprocessor, enclave, or privacy L2) varies in openness."
  p: "Strategy parameters, intent content, and order flow are hidden from the public. Aggregate on-chain state (AUM, performance) remains visible. A breach at the solver, key manager, or privacy substrate can expose strategy data wholesale."
  s: "Rides on the confidentiality of the substrate (Fully Homomorphic Encryption, enclave, or shielded execution), correct solver key management, and intent signature verification. Misconfigured access control is a common failure mode."

post_quantum:
  risk: medium
  vector: "Elliptic-curve signatures on intents (ECDSA, EdDSA) broken by CRQC. Encrypted intents and solver key hierarchies built on classical public-key encryption face HNDL risk."
  mitigation: "Migrate intent signatures to post-quantum signatures and encrypt long-lived intent archives under post-quantum KEMs as the ecosystem ships them. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-20, ERC-7573]

related_patterns:
  requires: [pattern-verifiable-attestation]
  composes_with: [pattern-private-shared-state-fhe, pattern-private-shared-state-tee, pattern-privacy-l2s, pattern-pretrade-privacy-encryption, pattern-dvp-erc7573, pattern-regulatory-disclosure-keys-proofs]
  see_also: [pattern-shielding, pattern-forced-withdrawal, pattern-private-transaction-broadcasting]

open_source_implementations:
  - url: https://github.com/zama-ai/fhevm
    description: "Fully Homomorphic Encryption runtime suitable as a private-vault substrate (testnet)"
    language: "Solidity, Rust"
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Privacy L2 usable as a programmable-privacy substrate for confidential strategy execution"
    language: Noir
---

## Intent

Allow institutional and DeFi actors to express trading or allocation intents that a vault executes privately: strategy parameters, order flow, and risk exposure stay hidden while deposited assets remain transparently custodied and redeemable. Auditors receive scoped disclosure; the public sees only aggregate state.

## Components

- Vault contract (ERC-20 wrapper) holds deposits transparently and exposes a redeem path for LPs and auditors.
- Encrypted intent: signed envelope containing target allocation, yield strategy, or hedging goal. Encrypted to the solver or to a confidential-computation substrate.
- Solver or strategy engine runs off-chain or inside a confidential runtime, converting intents into an execution plan.
- Confidential-computation substrate provides the privacy guarantee for the strategy. Options: Fully Homomorphic Encryption coprocessor, Trusted Execution Environment, or a privacy L2 with private notes.
- Execution layer: trades and allocations run on a privacy L2 or on a Fully Homomorphic Encryption runtime so order flow does not appear publicly.
- Audit and disclosure layer: viewing keys and attestation-logged scoped disclosure (EAS) let regulators and LPs verify execution authenticity without revealing strategy composition.
- Optional key-management module for encrypted strategy data and for solver credentials.

## Protocol

1. [user] Deposit assets into the vault; on-chain state records the deposit transparently.
2. [user] Submit a signed encrypted intent (for example, a target allocation or hedging rule) to the solver or substrate.
3. [prover] Solver or confidential runtime decrypts or partially computes the intent and produces an execution plan without exposing parameters.
4. [operator] Vault executes the plan privately on a privacy L2 or a Fully Homomorphic Encryption runtime; order flow stays hidden.
5. [contract] Settlement updates aggregate vault state on-chain; individual positions and strategy details remain private.
6. [auditor] Regulator or LP verifies correctness via a viewing key or an attestation-logged disclosure, without seeing strategy composition.

## Guarantees & threat model

Guarantees:

- Strategy logic, parameters, and order flow stay hidden from the public and from competitors.
- Vault asset balances and aggregate performance remain auditable on-chain.
- Intents are cryptographically signed; solvers cannot alter user intent without detection.
- Scoped disclosure lets regulators verify execution authenticity without revealing strategy composition.

Threat model:

- Confidentiality of the chosen substrate (Fully Homomorphic Encryption, enclave, privacy L2). A substrate breach exposes strategy data in bulk.
- Solver correctness and key hygiene. A compromised solver can leak intent content or front-run the execution plan.
- Misconfigured access control on viewing keys or attestation logs can turn scoped disclosure into public disclosure.
- Solver liveness. A single solver becomes a censorship chokepoint; multiple competing solvers mitigate.
- Timing and ordering metadata at the execution layer can leak strategy signals even when content is hidden; compose with private transaction broadcasting.

## Trade-offs

- Confidential computation adds latency and cost compared with transparent vault execution.
- Operational complexity: off-chain solver, encrypted intent pipeline, and privacy substrate together create a larger attack surface than a transparent vault.
- Auditor onboarding requires careful viewing-key or attestation scope design to avoid over-disclosure.
- Composability with public DeFi is limited while assets sit inside the privacy substrate; an exit step is required for public venues.

## Example

- A fund deposits a stablecoin into a private vault. It submits a signed encrypted intent: "allocate 60 per cent to an ETH yield strategy and 40 per cent to staked T-Bills." The solver decrypts the intent inside a Fully Homomorphic Encryption runtime, produces an execution plan, and the vault executes trades privately on a privacy L2. On-chain, only total AUM and aggregate performance are visible. An auditor later verifies correctness via an attestation-logged disclosure without seeing strategy composition.

## See also

- [Zama](../vendors/zama.md)
- [Fhenix](../vendors/fhenix.md)
- [Orion Finance](../vendors/orion-finance.md)
- [Aztec](../vendors/aztec.md)
- [Inco](../vendors/inco.md)
