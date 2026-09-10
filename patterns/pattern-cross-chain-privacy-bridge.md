---
title: "Pattern: Cross-Chain Privacy Bridge"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-06-18

works-best-when:
  - Moving assets between domains while minimizing linkability.
  - Both chains have shielded pools (required for full amount privacy).
  - Explicit bridge trust assumptions are acceptable.
avoid-when:
  - Both legs on same domain (use internal transfer).
  - Destination lacks a privacy primitive (no benefit to bridge privacy).
  - Regulatory requirements demand end-to-end public transparency.
  - Either or both chains have frequent reorgs (finality assumptions unreliable).

context: both
context_differentiation:
  i2i: "Institutions may operate their own relayers under bilateral agreements, reducing censorship risk but concentrating trust. KYC-aligned identities on both ends simplify audit, and escrow operators are typically known parties bound by contract. The anonymity set is narrower but acceptable because unlinkability from competitors is the primary goal."
  i2u: "End-users depend entirely on public relayer availability and cannot assume any operator will act in their interest. Permissionless relayer selection, source-side anonymization before bridging, and forced-withdrawal from destination pools are required to preserve meaningful privacy and liveness."

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Reaches `high` if relay infrastructure is decentralized with permissionless relayer selection and the destination mint cannot be gated by a single operator. Drops to `low` under custodial bridges where one operator controls mints."
  o: "Bridge contracts and verification circuits are often partially open. Improves to `yes` with reproducible builds and open-sourced bridge contracts, verification circuits, and relayer code."
  p: "Destination shielding provides receiver privacy, but source-side sender privacy is not automatic. Reaches `full` by mandating source-side anonymization (deposit through a pre-bridge shielded pool before generating commitments) and enforcing relayer-based submission with timing delays."
  s: "Varies widely by verification mechanism. ZK verification of source consensus with economic finality guarantees reaches `high`; custodial or MPC operators sit at `medium` or `low`."

post_quantum:
  risk: high
  vector: "ZK verification of source consensus inherits the underlying SNARK's PQ exposure. Shielded pool commitments and note encryption on the destination carry HNDL risk."
  mitigation: "STARK-based bridge proofs and hash-commitment shielded pools. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-20]

related_patterns:
  requires: [pattern-shielding]
  composes_with: [pattern-stealth-addresses, pattern-network-anonymity, pattern-regulatory-disclosure-keys-proofs]
  alternative_to: [pattern-permissioned-ledger-interoperability]
  see_also: [pattern-privacy-l2s, pattern-forced-withdrawal]

open_source_implementations: []
---

## Intent

Move assets between chains while preserving privacy on the destination by minting shielded notes whose ownership is not linkable to the source-domain depositor. Full sender and amount privacy requires privacy primitives on the source chain or pre-bridge anonymization; without these, the source lock transaction reveals the depositor address and amount.

## Components

- Source escrow contract locks assets on forward bridging or burns them on return. Emits an event carrying the destination commitment.
- Destination privacy primitive is a shielded pool with commitments and nullifiers. The bridge mints a note into this pool rather than transferring a transparent token.
- Cross-domain message and verification mechanism carries finality evidence from source to destination. Options include operator signatures, MPC or threshold signatures, optimistic claims with challenge windows, ZK succinct proofs of source consensus (zk-SPV), light clients, or TEE-attested enclaves.
- Relayer infrastructure submits finality proofs on the destination without linking the depositor. Fee model matters: if the depositor pays the relayer directly from a known address, the linkage is reintroduced.
- Optional compliance hooks include viewing keys for auditors, screening gates, and deposit limits.
- Timeout and recovery path on the source escrow allows the depositor to reclaim funds if the destination mint never completes.

## Protocol

1. [user] Generate a destination note commitment `C = hash(value, recipient_pubkey, randomness)`. The recipient is not revealed on the source chain.
2. [contract] Lock or burn the assets in the source escrow. Emit an event containing the amount, `C`, and a nonce. The lock is public on the source chain.
3. [relayer] Obtain finality evidence for the source lock: operator signature, threshold signature, optimistic claim, zero-knowledge proof of source consensus, light-client header, or TEE attestation, depending on the bridge's trust model.
4. [contract] The destination bridge contract verifies the finality evidence and mints a note into the shielded pool with commitment `C`.
5. [user] The recipient spends the note privately within the destination pool using standard shielded-pool semantics (nullifier reveal, proof of membership and ownership).
6. [user] For the return path, burn the shielded note on the destination, prove the burn to the source, and unlock the escrowed assets. The return leg can also be private if the source supports it.
7. [user] If the mint never completes within the timeout window, reclaim from the source escrow via the recovery path (requires a proof of non-mint or a governance override, depending on the bridge design).

## Guarantees & threat model

Guarantees:

- Destination recipient is not revealed on the source chain; the commitment hides their identity.
- Destination observers cannot link the recipient to the source depositor when a relayer submits the mint and timing is obfuscated. In custodial designs, the operator still knows the depositor but the destination chain does not.
- Amount privacy on the destination if the shielded pool supports confidential amounts or fixed denominations.
- Integrity: no double-mint (replayed deposits are blocked by nullifier or nonce tracking), and value is conserved across the two domains.
- Selective auditor access via viewing keys or compliance proofs, scoped to the destination pool's disclosure mechanism.

Threat model:

- Soundness of the verification mechanism. A compromised operator (custodial), a threshold breach (MPC), an unchallenged fraudulent claim (optimistic), a circuit bug (ZK), or a mishandled fork (light client) each break bridge integrity.
- Non-censoring relayer set. A single required relayer can refuse to submit the mint; the user then depends on the recovery path.
- Non-censoring destination sequencer or validator set for mint transactions.
- Reorg handling on the source chain. Finality assumptions must hold for the bridge's chosen finality evidence; chains with frequent reorgs break this.
- Source-side sender privacy is not provided by this pattern; depositor address and amount are visible on the source chain unless combined with pre-bridge anonymization.
- Metadata leakage (IP, timing, gas patterns, relayer fee payment) is out of scope for the bridge layer.

## Trade-offs

- Two-phase commit workflow, not instant atomic settlement. Latency depends on source finality and any challenge window.
- Cost scales with the verification mechanism. zero-knowledge proofs are expensive to generate; optimistic systems impose challenge delays; custodial designs are cheap but centralized.
- Reorg handling and cross-domain confusion (wrong chain ID, token mismatch) are recurring failure modes that must be guarded at the contract layer.
- Griefing through deposits that are never minted locks funds until timeout. The recovery path must be tested and well-documented.
- Key and governance risks: TSS or MPC signer collusion, view-key misuse, and malicious contract upgrades each sit outside the cryptographic trust model and require operational controls.
- Deployment topologies: pre-bridge mixing (deposit through a source-chain shielded pool first, then bridge: full sender privacy at added latency); hub-and-spoke (L1 as verification hub; multiple L2s prove deposits via the L1 bridge contract); privacy-to-privacy (shielded pools on both ends: privacy on both sender and receiver ends, more complex verification); asymmetric (one direction private, e.g. public L1 to private L2).

## Example

- An institution holds tokenized bonds on Ethereum L1 and wants to enable private secondary-market trading on a privacy L2.
- The institution deposits bond tokens into the source escrow, specifying destination commitment `C`. No recipient is revealed on L1.
- After L1 finality, an optimistic bridge posts a claim on the L2.
- After the challenge period, the L2 mints a shielded note for commitment `C` into its pool.
- The bond holder trades privately within the L2 shielded pool; counterparties and amounts are hidden from chain observers.
- On redemption, the holder burns the shielded note on the L2, proves the burn to L1, and unlocks the escrow.

## See also

- [ERC-7281: Sovereign Bridged Tokens (xERC20)](https://ethereum-magicians.org/t/erc-7281-sovereign-bridged-tokens/14979)
