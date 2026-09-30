---
title: "Pattern: Selective Disclosure (Viewing Keys + Zero-Knowledge Proofs)"
status: ready
maturity: testnet
type: standard
layer: hybrid
last_reviewed: 2026-09-29

works-best-when:
  - A regulator needs targeted visibility into specific trades or accounts without blanket transparency.
  - The workflow can require an approval step that is logged and revocable.
  - Zero-knowledge predicates can answer most regulator questions without releasing raw data.
  - A register of record (for example, a transfer agent or a securities registrar) must be able to rebuild beneficial ownership for any point in time.
avoid-when:
  - Policy requires fully public plaintext of all transactions.
  - The organisation cannot support the operational complexity of threshold key custody and audit storage.

context: both
context_differentiation:
  i2i: "Between institutions, regulator-initiated disclosure is a normal part of compliance. Both the subject and requester have audit obligations and legal recourse, and the approval workflow sits inside established supervisory channels."
  i2u: "For end users, disclosure should remain user-controlled wherever possible; see `pattern-user-controlled-viewing-keys`. When disclosure is imposed on users without their consent or knowledge the pattern collapses to `cr: low` because the institution can reveal user activity unilaterally."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Regulator-mandated disclosure is normal institutional operation and does not block transaction flow, so CR stays `medium`. Drops to `low` when disclosure is imposed on end users outside the institution's governance boundary. Threshold custody with independent operators can push CR toward `high` by preventing any single party from forcing a reveal."
  o: "Openness depends on the implementation. Pool-native viewing keys and open predicate circuits are inspectable; proprietary threshold KMS deployments are not. Disclosure policy code, mandate verification logic, and revocation flows should be auditable for the pattern to remain interoperable."
  p: "Disclosure is scoped: the regulator learns only what the mandate authorises. Zero-knowledge predicate responses leak no raw data. Viewing-key responses leak everything inside the key's scope, so key segmentation and time-boxing are important. The standing register mode discloses register data to the register of record by design, at a granularity the deployment sets; privacy against everyone else is unchanged."
  s: "Rides on threshold key custody, hardware-rooted policy engines, correct mandate parsing, and tamper-evident audit storage. Key compromise yields retroactive loss of privacy across the key's scope."

post_quantum:
  risk: high
  vector: "Viewing keys typically derive from EC-based key exchange and are stored as long-lived secrets; HNDL risk is high since ciphertexts and proofs can be archived now and broken later. Predicate circuits built on pairing-based proof systems are also affected."
  mitigation: "Migrate key-encapsulation to post-quantum KEMs (for example, ML-KEM) and move predicate circuits to STARK-based systems with hash commitments. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-shielding, pattern-user-controlled-viewing-keys, pattern-compliance-monitoring]
  alternative_to: [pattern-proof-of-innocence]
  see_also: [pattern-l2-encrypted-offchain-audit, pattern-private-pvp-stablecoins-erc7573, pattern-reproducible-audit-extraction, pattern-crypto-registry-bridge-ewpg-eas]

open_source_implementations:
  - url: https://github.com/ethereum-attestation-service/eas-contracts
    description: "Ethereum Attestation Service contracts, useful for logging disclosure grants"
    language: "Solidity"
---

## Intent

The pattern has two modes.

**Mandate-based disclosure** provides on-demand, scoped visibility into confidential trades and positions via threshold-controlled viewing keys or zero-knowledge predicate proofs that answer specific regulator questions. The institution keeps plaintext private by default and releases only the minimum information required to satisfy a specific, logged mandate.

**Standing register disclosure** serves a register of record such as a transfer agent or a securities registrar. The register receives a continuous feed sufficient to rebuild beneficial ownership for any point in time, without holder cooperation. This mode trades least privilege for completeness, so it suits only a party that is already entitled to the full register. How much the feed carries beyond ownership is a deployment choice; see Trade-offs.

## Components

- Threshold key management that holds viewing-key shares across independent operators and releases material only under policy.
- Policy engine that checks each request against the active mandate (jurisdiction, scope, time window, requester identity) before assembling a response.
- Predicate circuit library that can answer common regulator questions ("total volume in ISIN X on date Y"; "no trades with sanctioned parties in quarter Q") without releasing raw data.
- Attestation log that records every access grant as a signed, hashed entry; a public registry can anchor these hashes for tamper-evidence without revealing content.
- Approval workflow used by the institution's compliance team to review and sign off on requests before the policy engine acts.
- Register-of-record key (standing register mode): an encryption key held by the register of record, ideally under threshold custody. Every value-moving operation encrypts the resulting position data to it.

## Protocol

Mandate-based disclosure:

1. [regulator] Submit a scoped request specifying account, instrument, time window, and mandate.
2. [operator] The policy engine checks the request against the active mandate and approvals; an attestation logs the grant.
3. [operator] Assemble the response: either a time-limited viewing key reconstituted from threshold shares, or a zero-knowledge proof generated by the predicate circuit over the relevant private state.
4. [operator] Deliver the response to the regulator over an authenticated channel.
5. [auditor] Verify the disclosure record against the anchored hash to confirm that the response matches the approved mandate.

Standing register disclosure:

1. [operator] The register of record publishes its encryption key and anchors the key's hash in the attestation log.
2. [user] Every value-moving operation (issuance, transfer, redemption) encrypts the resulting position data, such as owner identifier and amount, to the register key.
3. [prover] The circuit constrains that ciphertext to encrypt the same values the proof commits to. Without this constraint, completeness rests on the holder's legal obligation to encrypt honestly.
4. [contract] The ciphertext is published with the transaction, so the feed needs no separate delivery channel.
5. [operator] The register of record decrypts the feed, rebuilds beneficial ownership for any point in time, and reconciles it against its off-chain records.

## Guarantees & threat model

Guarantees:

- Least-privilege, revocable access: viewing keys are time-boxed; predicate proofs leak no raw data.
- Full audit trail: every grant is recorded, signed, and anchored.
- Predicate proofs can answer common compliance questions without exposing transaction content.
- Standing register disclosure: the register of record can rebuild beneficial ownership for any point in time without holder cooperation, provided the circuit constrains the ciphertext.

Threat model:

- Threshold key custody integrity. A coalition of operators above the threshold can forge responses or leak keys.
- Mandate parsing correctness. A bug in the policy engine can widen scope beyond what the mandate authorises.
- Attestation log availability. If the log is rewritable or unobserved, disclosures can be retroactively denied.
- Predicate circuit soundness and input binding. A misbound predicate may answer a different question than the one logged.
- Register key compromise (standing register mode) exposes the entire register for every period the feed covers. Threshold custody and key rotation carry more weight than in the mandate-based mode.
- Unconstrained ciphertexts. If the circuit does not bind the register ciphertext to the committed values, a holder can publish data the register cannot decrypt, and completeness falls back to legal enforcement.
- Side channels in custody infrastructure are out of scope for this pattern.

## Trade-offs

- Operational complexity: threshold key custody, rotation, and incident response need dedicated runbooks.
- Predicate authoring requires discipline: circuits must mirror mandate semantics exactly, and changes need auditable version control.
- Response latency increases with threshold reconstitution or proof generation time; batch workflows absorb this better than real-time supervisory queries.
- Regulator tooling maturity varies: some supervisory authorities still require raw data formats, limiting applicable use cases.
- The standing register mode inverts least privilege, and feed granularity has no free option. Per-operation data lets the register rebuild holdings for any date, and with them most flows, since two holders' balances changing together reveal the pair. Coarser feeds, such as periodic snapshots, reveal less but may not meet record-keeping duties. What a register of record must minimally see is an open question. Constraining the ciphertext in-circuit also adds proving cost to every value-moving operation.

## Example

A supervisory authority asks for trades on a given date in a specific instrument. The policy engine matches the request against the institution's active mandate, an approval record is logged, and a 24-hour viewing key reconstituted from threshold shares is issued. The regulator reviews the trades during the window; at expiry the key is revoked automatically, and the attestation log retains a hashed record for future audit.

The transfer agent of a tokenized security holds the register key. Each issuance, transfer and redemption publishes position data encrypted to that key, bound to the proof by the circuit, so the register can reproduce holder records, eligibility status and tax reports for any date without asking holders.

## See also

- [EAS documentation](https://easscan.org/docs)
- [Aztec](../vendors/aztec.md)
