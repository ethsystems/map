---
title: "Pattern: Compliance Monitoring"
status: ready
maturity: concept
type: standard
layer: hybrid
last_reviewed: 2026-09-30

works-best-when:
  - Institution must monitor transactions for AML or sanctions compliance.
  - Privacy is required but regulators need audit capability.
  - Real-time screening is mandatory before settlement.
avoid-when:
  - Full transparency is acceptable (no privacy requirement).
  - Jurisdiction has no transaction monitoring requirements.

context: both
context_differentiation:
  i2i: "Counterparties may accept a single compliance oracle operated by a trusted consortium member or shared service. Bilateral contracts govern screening rules and dispute handling. Appeals are resolved through existing institutional channels."
  i2u: "End-users have no visibility into oracle decisions and no direct recourse if flagged incorrectly. Threshold consensus across independent oracle operators, transparent appeal processes, and ZK-based screening that hides transaction details from individual operators become essential."

crops_profile:
  cr: none
  o: partial
  p: partial
  s: low

crops_context:
  cr: "The screening oracle is a single point of censorship by design. Reaches `medium` if replaced with a threshold consensus oracle network that includes appeal routes and cannot unilaterally block transactions."
  o: "Rule engines are often proprietary and bundled with screening vendors. Improves to `yes` by open-sourcing the rule engine under a copyleft license and making rule updates auditable."
  p: "The oracle sees some transaction metadata by design. Reaches `full` when enforced screening uses zero-knowledge proofs (prove amount under threshold, prove counterparty not on sanctions list) so the oracle validates attestations without seeing amounts or identities."
  s: "Rides on a single operator's honesty and availability. Improves to `high` by replacing single-operator trust with threshold KMS and timelocked recovery for rule updates."

post_quantum:
  risk: medium
  vector: "Signatures on screening attestations inherit host-chain and oracle signature assumptions. zero-knowledge proofs used in privacy-preserving screening inherit their proof system's exposure."
  mitigation: "Hash-based signatures and STARK-based screening proofs. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [EAS]

related_patterns:
  composes_with: [pattern-regulatory-disclosure-keys-proofs, pattern-user-controlled-viewing-keys, pattern-verifiable-attestation, pattern-zk-kyc-ml-id-erc734-735, pattern-erc3643-rwa]
  see_also: [pattern-shielding, pattern-proof-of-innocence, pattern-zk-promises]

open_source_implementations: []
---

## Intent

Enable institutions to screen private transactions for regulatory compliance (AML, sanctions, fraud) without exposing transaction details to unauthorized parties. Balance privacy preservation with auditability through selective screening approaches and tiered disclosure, so settlement can proceed under compliance controls while counterparty identities and amounts remain shielded from public view.

This is an orchestration pattern that composes primitives (viewing keys, zero-knowledge proofs, threshold KMS, attestations) into a compliance workflow. The unique contribution is the rule engine, alert pipeline, and audit trail that hold the workflow together; the underlying disclosure primitives are linked via `related_patterns`.

## Components

- Compliance oracle or screening service evaluates transactions against sanctions lists, AML rules, and internal policies. Can be centralized, federated, or threshold-operated.
- Rule engine stores jurisdiction-specific rules with versioning. Rule updates are logged and auditable.
- Threshold key management issues and rotates viewing keys distributed to authorized parties, so no single operator can unilaterally decrypt transaction contents.
- zero-knowledge proof verifiers (optional) validate compliance attestations such as "amount below reporting threshold" or "counterparty not on sanctions list" without revealing the underlying values.
- Alert and case management system handles flagged transactions through severity tiers with defined response times.
- Audit log records every screening decision with timestamp, rule version, and decision hash. Typically anchored on-chain via attestations for tamper evidence.

## Protocol

1. [user] Sender constructs a transaction and generates a compliance attestation (viewing key or zero-knowledge proof) alongside the encrypted payload.
2. [operator] Compliance oracle pre-screens the recipient against sanctions lists and internal policies before the transaction is submitted.
3. [operator] Oracle verifies the transaction against AML rules using the attestation, without seeing the full plaintext when zero-knowledge proofs are used.
4. [operator] If screening passes, the oracle emits a cleared attestation; if flagged, an alert is generated with severity level.
5. [operator] High-severity alerts trigger a hold and manual review; low-severity cases are logged for batch review.
6. [contract] Cleared transactions settle on the host chain; flagged transactions remain held pending resolution.
7. [auditor] Periodic compliance reports are generated from the audit log for regulator filing. Regulators can request selective disclosure via viewing keys on demand.

## Guarantees & threat model

Guarantees:

- Transaction details are visible only to the compliance function and authorized parties.
- All transactions are screened against current sanctions lists and jurisdiction rules before settlement.
- Screening decisions produce an immutable, timestamped audit trail suitable for regulator review.
- Counterparty identities are protected from public view; disclosure is scoped to authorized auditors via viewing keys.

Threat model:

- Soundness of any zero-knowledge proofs used for screening attestations.
- Non-colluding oracle operators. A single compromised or coerced operator in a centralized deployment can leak transaction metadata or unilaterally block transactions.
- Rule engine integrity. A tampered rule set can allow prohibited transactions or block legitimate ones.
- Key management for viewing keys. Compromised keys enable unauthorized decryption of historical transactions.
- Network-layer metadata (IP, timing, submission patterns) is out of scope and must be covered separately.

## Trade-offs

- Real-time screening adds latency to each transaction, typically tens to hundreds of milliseconds depending on implementation and ruleset. Batch screening for low-risk flows mitigates this.
- False positives from overly strict rules block legitimate transactions. Tiered thresholds and human review for edge cases are standard mitigations.
- Oracle trust is a concentration point. Threshold operation with multiple providers or TEE-based screening reduces this risk.
- Per-transaction screening fees can be significant at volume; internal screening for high-frequency flows is common.
- Centralized oracles retain unilateral censorship power even when privacy is otherwise strong; threshold governance and transparent appeals are necessary counterweights.

## Example

- An institution initiates a large bond purchase on a privacy L2.
- The sender's compliance node pre-screens the counterparty against the sanctions list (clear).
- The transaction is submitted with an encrypted amount and a zero-knowledge proof that the amount is within the reporting threshold.
- The oracle verifies the proof, checks the counterparty risk score (medium), and clears the transaction.
- Settlement executes. The audit log records timestamp, screening version, result, and proof hash.
- A monthly report aggregates cleared transactions for the regulatory filing.

## See also

- [FATF Travel Rule guidance](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-rba-virtual-assets-2021.html)
- [EAS specification](https://docs.attest.org/)
