---
title: "Pattern: Low-cost L2 + Off-chain Encrypted Audit Log"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-09-18

works-best-when:
  - You need hidden amounts and positions with a minimal on-chain footprint.
  - Cheap daily settlement is preferable to full on-chain private compute.
  - A trusted off-chain operator or consortium can run the encrypted log.
avoid-when:
  - The regulator requires full on-chain plaintext of each transaction.
  - Off-chain infrastructure cannot be operated reliably across multiple regions.

context: both
context_differentiation:
  i2i: "Between institutions, the encrypted log can be hosted by a consortium or mutualized operator. Both parties have legal recourse if log integrity is disputed, and the on-chain Merkle anchor constrains what can be retroactively changed. Scoped regulator keys cover audit without revealing unrelated trades."
  i2u: "The user cannot verify that an off-chain log contains their own entries unless they keep client-side ciphertext and can reproduce commitments. Without a forced on-chain settlement path, the user depends on the operator to release data needed for withdrawal. The pattern should therefore surface an L1 escape hatch and give the user a copy of their encrypted records."

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Reaches `high` only when the L2 exposes an L1 escape hatch usable without operator cooperation. On sequencer-controlled L2s with no force-inclusion path, censorship resistance is effectively `low`."
  o: "On-chain contracts can be open source; the threshold KMS and off-chain log code are often closed. Openness improves when the KMS software is released and multiple implementations can run the log."
  p: "Plaintext is off-chain; commitments and access logs on-chain leak timing and counterparty pairs. Access-pattern privacy requires additional measures (proxy re-encryption, mixnet routing for log reads)."
  s: "Settlement integrity rides on the host L2 and the periodic Merkle anchor. Disclosure integrity rides on the threshold KMS quorum and retention of encrypted records across regions."

post_quantum:
  risk: medium
  vector: "Symmetric record encryption (AES-GCM) is PQ-safe; key wrapping under EC-based threshold schemes is broken by CRQC, with HNDL risk for long-retention archives."
  mitigation: "Rotate wrapped keys using ML-KEM or hash-based threshold schemes before CRQC arrival. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [ERC-7573, EIP-4844, EAS]

related_patterns:
  composes_with: [pattern-dvp-erc7573, pattern-regulatory-disclosure-keys-proofs, pattern-forced-withdrawal]
  alternative_to: [pattern-private-shared-state-fhe, pattern-shielding]
  see_also: [pattern-modular-privacy-stack, pattern-commit-and-prove]

open_source_implementations:
  - url: https://github.com/ethereum-attestation-service/eas-contracts
    description: "Ethereum Attestation Service reference contracts for access-log attestations"
    language: Solidity
---

## Intent

Run settlement on a low-cost L2, publish only commitments and hashes on chain, and keep the full transaction facts in an append-only encrypted log off chain. Integrity is anchored by periodic Merkle roots submitted on chain. Regulators and auditors receive scoped decryption keys or predicate proofs. Delivery-versus-payment is expressed through an atomic settlement standard.

## Components

- On-chain audit contract that accepts `AuditCommit(bytes32)` entries and records hourly Merkle roots over the off-chain log.
- Append-only encrypted log, replicated across regions, storing per-trade records keyed by a content address.
- Per-trade symmetric key, wrapped to a threshold set of authorities so that disclosure requires a quorum rather than a single custodian.
- Atomic settlement contract implementing cross-leg delivery-versus-payment over cash and asset legs.
- Access-logging attestations emitted on chain whenever a scoped key is issued or used.

## Protocol

1. [user] Negotiate and match the trade off chain; optionally encrypt the routing metadata.
2. [operator] Write the encrypted record to the log, compute its commitment, and submit `AuditCommit` on chain.
3. [operator] Aggregate the window's commitments into a Merkle root and anchor it on chain at the configured cadence.
4. [contract] Escrow both legs and finalize atomically through the delivery-versus-payment contract.
5. [regulator] Receive a scoped decryption key or predicate proof for a specific record; the issuance is logged through an on-chain attestation.
6. [auditor] Replay the log against the anchored roots to confirm that no record has been rewritten after the fact.

## Guarantees & threat model

Guarantees:

- Public observers see only commitments and hashes; amounts, identities, and positions remain off chain.
- Merkle anchoring makes the log tamper-evident: any silent rewrite breaks the on-chain root.
- Atomic delivery-versus-payment prevents one-sided settlement failure.
- Disclosure is scoped and logged, so access is auditable after the fact.

Threat model:

- Trust in operator availability and retention of the encrypted log; loss of ciphertext cannot be recovered from the chain alone.
- Threshold quorum of the key-wrapping authorities; a colluding quorum can decrypt records outside the disclosure process.
- Non-censoring sequencer on the host L2. Without a usable L1 escape hatch, a censoring sequencer can block settlement and audit commits.
- Access-pattern and timing side channels on the log remain visible to anyone hosting or monitoring the storage layer.

## Trade-offs

- Operational overhead of running redundant encrypted storage across regions with retention and rotation policies.
- Key governance cost: rotating wrapped keys and re-encrypting archived records is non-trivial at scale.
- Cross-region replication and KMS coordination add latency to disclosure flows.
- Failure mode: log rewrite attempt is detected by Merkle reconciliation but recovery still requires access to earlier ciphertext; multi-region backups are the mitigation.

## Example

A dealer sells a bond to an asset manager on the L2. The chain records only the commitment and the hourly Merkle root; full trade details sit encrypted in the log. Delivery-versus-payment finalizes atomically on chain. The national supervisor later receives a 24-hour scoped key for that record, and the issuance is attested on chain so the disclosure is itself auditable.

## See also

- [ERC-7573 spec](https://ercs.ethereum.org/ERCS/erc-7573)
- [EIP-4844 (blobs)](https://eips.ethereum.org/EIPS/eip-4844)
- [EAS docs](https://easscan.org/docs)
