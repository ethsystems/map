---
title: Private Registry
status: ready
primary_domain: Identity & Compliance
secondary_domain: Funds & Assets
---

## 1) Use Case

Operating authoritative registries (securities holders, business entities, licenses, environmental permits) on-chain where registry entries must be verifiable by authorized parties but the full registry state must not be publicly exposed. Distinct from authentication (which proves membership in a registry); this use case is about institutions that *run* registries: maintaining records, processing updates, and enabling cross-border lookups, all with privacy.

## 2) Additional Context

## 3) Actors

Registry operators (governments, regulators, CSDs) · Registered entities (companies, holders, licensees) · Verifiers/queriers (financial institutions, employers, customs) · Regulators · Auditors

## 4) Problems

### Problem 1: Operating a Registry Where Entries Are Private but Verifiable

Registry operators must maintain authoritative records where individual entries (holder positions, entity status, license validity) are verifiable by authorized parties without exposing the full registry to all participants. A securities registry operator, for example, must prove a holder exists to a regulator without exposing the full holder list.

**Requirements:**

- **Must hide:** Full registry state, individual entry details from unauthorized parties, relationships between entries
- **Public OK:** Registry existence, schema, aggregate statistics (total registered entities, etc.)
- **Authorized access:** Merkle proofs for specific entry lookups; scoped viewing keys for regulators; batch verification for auditors

**Constraints:**

- Registry must support efficient updates (additions, modifications, revocations) without rebuilding the entire proof structure
- Query latency requirements vary by use case (securities settlement needs near-real-time; business registry can tolerate minutes)
- Legal requirements for registry completeness and accuracy (registries of record)

### Problem 2: Cross-Border Registry Interoperability With Privacy

Institutions need to query foreign registries (is this company registered in jurisdiction X? is this license valid?) without bulk data exchange. Current cross-border verification often requires bilateral agreements and full data sharing, creating privacy and sovereignty concerns.

**Requirements:**

- **Must hide:** Query patterns (which entities are being checked), full foreign registry contents
- **Public OK:** Registry availability, supported query types, attestation format
- **Regulator access:** Cross-border query audit trail for mutual legal assistance

**Constraints:**

- Different jurisdictions use different identifiers, schemas, and legal frameworks
- Data sovereignty: registry data must not leave jurisdictional control
- Latency: some cross-border checks (customs, KYB) are time-sensitive

### Problem 3: Tamper-Evident Change Control

Registry updates (new entries, status changes, revocations) must be auditable without exposing what changed to unauthorized parties. Regulators and auditors need to verify that updates followed proper authorization, but the content of updates may be commercially or personally sensitive.

**Requirements:**

- **Must hide:** Specific change details from unauthorized parties, identity of entities involved in updates
- **Public OK:** Update frequency, registry version/epoch, proof of proper authorization
- **Auditor access:** Full change history for specific entries on demand; proof that all updates were authorized

**Constraints:**

- Append-only audit trail with privacy for change content
- Revocation must propagate to relying parties without revealing which entry was revoked (where applicable)
- Long retention periods (registries may need decades of history)

## 5) Recommended Approaches

Approach TBD. Key architectural considerations:

- Merkle-tree-based registries: entries stored as leaves, proofs for individual lookups without revealing siblings
- Encrypted state with scoped decryption: registry state encrypted, viewing keys scoped by role (operator, regulator, entry owner)
- Cross-border attestation bridges: foreign registry queries answered with attestations, not raw data

## 6) Open Questions

- What is the right data structure for privacy-preserving registries that support efficient updates (sparse Merkle trees, indexed Merkle trees, append-only logs)?
- How do privacy-preserving registries interact with existing registry legal frameworks (e.g., eWpG crypto-registry requirements)?
- What is the minimum viable cross-border registry query protocol that satisfies both privacy and sovereignty requirements?
- How to handle registry corrections (errors, court orders) in an append-only privacy-preserving system?
- What trust assumptions must registered entities make about the registry operator (liveness, censorship resistance, Data Availability, honest state transitions), and how can these be minimized or made verifiable?

## 7) Notes And Links

- Related patterns: [Crypto-Registry Bridge (eWpG/EAS)](../patterns/pattern-crypto-registry-bridge-ewpg-eas.md), [Verifiable Attestation](../patterns/pattern-verifiable-attestation.md), [Private MTP Auth](../patterns/pattern-private-mtp-auth.md)
- Related use cases: [private-identity.md](private-identity.md) (consuming registries for authentication; this use case is about operating them)
- See also: [EPIC map](https://epic-webapp.vercel.app/) (GovTech & EPIC team, demo data): business registry, land registry, environmental registries, licensing
