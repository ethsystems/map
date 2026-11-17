---
title: "Approach: Private Registry"
status: draft
last_reviewed: 2026-11-15

use_case: private-registry
related_use_cases: [private-identity]

primary_patterns: [pattern-verifiable-attestation, pattern-regulatory-disclosure-keys-proofs]
supporting_patterns: [pattern-crypto-registry-bridge-ewpg-eas, pattern-private-information-retrieval]
---

# Approach: Private Registry

## Problem framing

### Scenario

A securities registry operator maintains holder positions for an issuer whose shareholder list is commercially sensitive. A regulator must be able to confirm any holder's position on demand, a bank must verify a license is valid before onboarding a counterparty, and a customs authority abroad must check an entity's registration without either side bulk-exporting registry data. The operator needs efficient updates and revocations, decades of retention, and a change trail auditable without exposing what changed. Registry existence, schema and aggregates are public; entry contents, relationships and query patterns are not.

### Requirements

- Prove specific entries (positions, licenses, entity status) to authorized parties only
- Keep the full registry state and entry relationships hidden
- Support efficient updates and revocations without rebuilding proof structures
- Answer cross-border queries without bulk data export out of jurisdiction
- Give regulators and auditors scoped, on-demand access to entries and change history
- Preserve an append-only, tamper-evident change trail across long retention

### Constraints

- Registries of record carry legal completeness and accuracy obligations
- Latency varies by use case: settlement wants near-real-time, business registries can wait minutes
- Jurisdictions use different identifiers, schemas and legal frameworks
- Revocations must propagate to relying parties without always revealing which entry was revoked
- Registered entities must be able to rely on the operator's liveness and correct state transitions
- Legal forms constrain designs: eWpG section 16(1) requires a forgery-proof recording system, defined in section 4(11) as a decentralized one, so a single operator that keeps a tree and anchors its root does not obviously qualify ([de-eWpG](jurisdictions/de-eWpG.md))

## Approaches

The first two sub-approaches are the real alternatives for the domestic registry; the third is a cross-border layer that composes over either of them.

### Sparse Merkle registry with selective proofs

```yaml
maturity: documented
context: both
crops: { cr: low, o: yes, p: partial, s: medium }
uses_patterns: [pattern-verifiable-attestation, pattern-regulatory-disclosure-keys-proofs]
```

**Summary:** Registry entries live as leaves in a sparse Merkle tree anchored on-chain; verifiers receive Merkle proofs for specific lookups without seeing the rest of the tree.

**How it works:** The operator maintains the sparse Merkle tree, whose root is anchored on-chain each epoch. A verifier asking "is this entity registered and in good standing" receives a Merkle proof for that leaf, which proves the entry's inclusion and current status without revealing sibling entries. Updates touch only affected paths, so additions, modifications and revocations stay cheap even in very large registries. Revocations update the leaf's status field, and non-existence proofs let a verifier establish that no entry exists for a key. Leaves must be salted commitments: a Merkle path carries a sibling hash at every level and empty subtrees hash to known defaults, so an unsalted proof shows which subtrees near the key are occupied and a low-entropy leaf can be brute-forced from its hash. Where the path itself must stay hidden, wrap the proof in zk as in `pattern-private-mtp-auth`.

**Trust assumptions:**
- The operator maintains the tree honestly against the anchored root
- Anchoring cadence bounds how stale a proof can be
- Verifiers trust the anchoring chain's settlement guarantees

**Threat model:**
- An operator anchors a root that contains an unauthorized change, or shows different roots to different verifiers (mitigated by validity proofs over state transitions and a single canonical anchor)
- Proof granularity leaks structure through repeated related queries
- Operator liveness: a stalling operator freezes verifiability

**Works best when:**
- Individual lookups, not bulk access, are what verifiers need
- The registry is large and update efficiency matters

**Avoid when:**
- Verifiers need to enumerate or join across entries
- The operator itself is the trust concern

CROPS rationale: CR low, because the operator gates who may query and no independent path bypasses that approval; proofs answer only what a verifier asks, but admission is institutional. O yes: sparse Merkle constructions and anchoring are standard, open technology. P partial: individual entries stay hidden, while query patterns and aggregate structure remain observable. S medium: cryptographic soundness is strong, with operator liveness and honest maintenance the residual trust.

### Encrypted registry state with scoped viewing keys

```yaml
maturity: documented
context: both
crops: { cr: low, o: partial, p: partial, s: medium }
uses_patterns: [pattern-regulatory-disclosure-keys-proofs]
```

**Summary:** The registry's state is stored encrypted; role-scoped viewing keys let the operator, regulators and entry owners decrypt exactly their slice without exposing the rest.

**How it works:** Every entry is written to the registry in encrypted form, with the on-chain record carrying commitments and version information rather than plaintext. Viewing keys are issued per role and per scope: an entry owner can read its own entry, a regulator can read the entries in its jurisdiction or investigation, and the operator can read what it operates. Auditors receive derived, scoped keys for a defined period rather than standing access. A time window limits which data a key decrypts, not how long the holder can read it: revoking a key stops future grants but cannot recall data already in scope. Change history stays encrypted, so the trail is verifiable in structure while its content stays gated.

**Trust assumptions:**
- Key management is sound: a leaked regulator key exposes its whole scope
- The encryption layer and its implementations are open and audited
- The operator writes state transitions honestly, whether or not it can read them

**Threat model:**
- Key compromise reveals an entire scope, not just one entry
- Traffic analysis on update frequency correlates registry activity with external events
- Scope creep: keys issued for one investigation reused for another

**Works best when:**
- Entries are genuinely sensitive at the data layer, not just the query layer
- Regulators accept scoped, time-boxed keys instead of standing access

**Avoid when:**
- Key ceremony and recovery overhead outweigh the registry's sensitivity
- Jurisdictions mandate plaintext mirroring of the registry of record

CROPS rationale: CR low, because access is permissioned by operator key issuance with no independent path. O partial: the cryptography is open, while per-registry key schemes are deployment-specific. P partial: nothing but ciphertext and commitments is public at the data layer, but the operator can read what it operates, so visibility against the operator is partial, and regulator keys narrow but do not remove it. S medium: key management concentration is the dominant risk.

### Cross-border attestation bridge (overlay)

```yaml
maturity: concept
context: i2i
crops: { cr: none, o: partial, p: partial, s: medium }
uses_patterns: [pattern-crypto-registry-bridge-ewpg-eas, pattern-verifiable-attestation, pattern-private-information-retrieval]
```

**Summary:** Foreign registry queries are answered with signed attestations rather than raw data exchange, so registry data stays in its jurisdiction. The answering operator learns which entity is checked unless the lookup uses private information retrieval, and on-chain attestations expose query timing and volume to anyone.

**How it works:** Each registry operator runs an attestation endpoint that answers defined queries ("is this license valid", "is this entity registered") with a signed, schema-defined attestation anchored on-chain. A querying institution in another jurisdiction verifies the attestation against the operator's public key and the anchoring chain, without ever receiving bulk data. Attestations anchored individually expose the timing and volume of checks; anchoring batched roots, or issuing attestations off-chain with batched on-chain proofs, keeps that leak down, and a PIR lookup hides which entity was checked. The query audit trail lives with the two parties and is disclosed to authorities through scoped disclosure when mutual legal assistance requires it.

**Trust assumptions:**
- Each operator attests honestly about its own registry
- Attestation schemas are agreed bilaterally or through a standards body
- Attestation validity windows bound staleness

**Threat model:**
- An operator attests about stale or altered state
- Query-volume analysis infers which entities are under review
- Schema divergence makes attestations non-comparable across jurisdictions

**Works best when:**
- Bilateral or multilateral verification is recurring and well-defined
- Data sovereignty rules forbid bulk export

**Avoid when:**
- The query needs rich data, not a yes/no or attribute answer

CROPS rationale: CR none, because the bridge pattern itself has no censorship-resistance story: admission is bilateral and either operator can stop answering. O partial: the bridge pattern and attestation layers are open, but the middleware is often proprietary and the pattern covers one German registrar mirroring record hashes, with no cross-border query flow. P partial: answers are scoped to the question, while query existence and volume are visible to the counterpart unless the lookup uses PIR. S medium: attestation honesty and schema governance are the trust carriers.

## Comparison

| Axis | Sparse Merkle registry with selective proofs | Encrypted state with scoped viewing keys | Cross-border attestation bridge |
|---|---|---|---|
| **Maturity** | documented | documented | concept (overlay) |
| **Context** | both | both | i2i |
| **CROPS** | CR low · O yes · P partial · S medium | CR low · O partial · P partial · S medium | CR none · O partial · P partial · S medium |
| **Trust model** | honest operator, anchored root | key management per role | honest attesting operators |
| **Privacy scope** | entries hidden; query patterns visible | data fully encrypted; keys gated | answers scoped; volume visible |
| **Performance** | cheap per-lookup proofs, cheap updates | encryption on the write and read paths | one attestation per query |
| **Operator req.** | Yes (tree maintenance) | Yes (key issuance) | Yes (attestation endpoint) |
| **Cost class** | low | medium | low |
| **Regulatory fit** | depends on jurisdiction; see [eWpG](jurisdictions/de-eWpG.md) | strong where scoped keys are accepted: regulator access without bulk mirroring | strong where answers are attestations rather than raw exchange: sovereignty holds by construction |
| **Failure modes** | operator liveness; stale proofs | key compromise; scope creep | stale attestations; schema drift |

## Persona perspectives

### Business perspective

A registry operator's product is trust in its records, and the three models differ in what that trust costs to run. The sparse Merkle design is the least invasive and keeps the operator's role ordinary: maintain the tree, anchor roots, answer proofs. Encrypted state is worth its key-management overhead only when entry contents themselves are the sensitive asset, as with securities holders or personal licenses. The attestation bridge is the model that unlocks cross-border revenue: it answers foreign queries without breaching the sovereignty rules that otherwise block them, and it composes with either domestic model underneath. Most operators will start domestic and add the bridge as cross-border demand arrives.

### Technical perspective

The sparse Merkle registry is well-trodden: non-existence proofs, cheap updates and anchored roots are all standard, and the engineering risk is cadence and tooling, not cryptography. Encrypted state pulls key management into the registry's core, and revocation semantics get subtle: revoking a key is easy, revoking an entry without revealing which one needs care. The attestation bridge is the lightest to build and the most governance-heavy to keep schemas aligned across jurisdictions. Retention of decades of history is easiest in the first model, where old roots and proofs stay small, and most demanding in the second, where encrypted archives need durable key custody.

### Legal & risk perspective

Registries of record carry statutory completeness and accuracy duties, so a privacy-preserving design must still satisfy whatever the jurisdiction requires about the registry's authoritative form; an encrypted registry may need a defined reconciliation with the legal record. Revocation propagation has legal consequences for relying parties, and the window between revocation and relying-party awareness should be explicit in the design. Cross-border attestations sit inside mutual legal assistance frameworks: the scoped disclosure path should be designed with the authorities' actual request procedures in mind, and errors or court-ordered corrections need a defined path in an append-only structure. This is not legal advice.

## Recommendation

### Default

For the baseline registry operator, the sparse Merkle registry with selective proofs is the default: it delivers verifiable lookups and cheap updates with the smallest departure from conventional registry engineering, and regulator access arrives through scoped disclosure on the same trail.

### Decision factors

- If entry contents themselves are the sensitive asset, use encrypted state with scoped viewing keys.
- If cross-border queries are the binding constraint, add the attestation bridge on top of the domestic model.
- If the operator's own honesty is the main trust concern, neither Merkle nor encrypted state removes it; distribute the operator role instead.

### Hybrid

Run the domestic registry as a sparse Merkle tree for lookups, encrypt the entries that are genuinely sensitive with scoped viewing keys, and expose cross-border verification through the attestation bridge over the same anchored roots.

## Open questions

1. What data structure best supports privacy-preserving registries with efficient updates at national scale (sparse Merkle trees, indexed Merkle trees, append-only logs)?
2. How do privacy-preserving registries interact with existing registry legal frameworks such as the eWpG crypto-registry requirements ([de-eWpG](jurisdictions/de-eWpG.md))?
3. What is the minimum viable cross-border attestation protocol that satisfies both privacy and sovereignty requirements?
4. How should registry corrections (errors, court orders) work in an append-only privacy-preserving system?
5. What trust assumptions must registered entities make about operator liveness, censorship resistance, Data Availability and honest state transitions, and how can they be minimized or made verifiable?
