---
title: "Vendor: The Graph"
status: ready
website: https://thegraph.com
category: infrastructure
ethereum_aligned: true
last_reviewed: 2026-08-05
maturity: production
---

# The Graph – Firehose, Substreams, Subgraphs (decentralized indexing and query infrastructure)

## What it is

An open source indexing stack. 76 networks are supported for subgraph indexing; on 23 of them (18 mainnets, including Ethereum) subgraphs are served by a decentralized network of independent, GRT-staked indexers. The Graph indexes public chain state and is not a privacy technology. Its role in privacy architectures is the read and audit path: deterministic, re-executable extraction and serving of the public artifacts confidential systems emit (commitments, nullifiers, anchored roots, attestation logs), so auditors and regulators can check disclosures against an independently recomputable record of on-chain emissions.

## Fits with patterns

- [Pattern: Reproducible Audit Extraction](../patterns/pattern-reproducible-audit-extraction.md)
- [Pattern: Compliance Monitoring](../patterns/pattern-compliance-monitoring.md)
- [Pattern: Low-cost L2 + Off-chain Encrypted Audit Log](../patterns/pattern-l2-encrypted-offchain-audit.md)
- [Pattern: Selective Disclosure (Viewing Keys + Zero-Knowledge Proofs)](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Pattern: Verifiable Attestation](../patterns/pattern-verifiable-attestation.md)
- [Pattern: Private Information Retrieval](../patterns/pattern-private-information-retrieval.md)

## Not a substitute for

- Privacy layers (shielded pools, privacy L2s, FHE or TEE execution); it adds no confidentiality.
- Reserve or solvency attestation; off-chain assets cannot be proven from chain data.
- In-zone operator privacy on permissioned ledgers with no public state to index.
- Price-feed oracles; it serves chain-derived facts, never off-chain observations.
- KYT screening and sanctions intelligence.

## Architecture

- **Subgraphs** are the indexing data service: mappings turn contract events into a typed entity store queried over GraphQL. On the decentralized network since 2020: independent indexers serve each deployment with proofs of indexing and signed response attestations.
- **Substreams** is the streaming transform engine: deterministic, composable Rust modules over raw block data with high-throughput parallel backfill, feeding SQL, file, and custom sinks. In production since 2023, run self-hosted or bought from a hosted operator. It is not yet a data service on the decentralized network, so a Substreams result carries no indexer attestation and no dispute path. Determinism is its trust mechanism: anyone can re-run it and compare. The network supplies a different one, attributability, where an indexer stakes collateral against the answer it signs. Subgraphs on the network have both.
- **Firehose** is the extraction layer beneath Substreams, also usable by other indexing systems: full chain history captured as deterministic, content-addressed flat files surviving execution-layer history pruning (EIP-4444); open source tooling recomputes receipt and transaction roots from extracted data and proves pre-Merge block inclusion against Ethereum's canonical header accumulator.
- **Decentralized network:** publishing is permissionless; indexers stake GRT and serve queries for fees. Paid responses carry EIP-712 signed attestations binding request and response hashes to the indexer's staked allocation; conflicting attestations ground an on-chain dispute settled by arbitration with slashing. Attestations are signatures, not validity proofs: responses become non-repudiable and slashable.
- **Gateways:** optional open source routing; anyone can operate one; clients can query indexers directly or pin one to cross-check.
- **Hosted delivery** is the common enterprise path: a single operator runs Firehose, Substreams, or subgraph APIs under contract. Open source core, but deployment, billing and API layers are proprietary; one operator can exclude a customer and attestations become contractual terms. The self-hosting exit stays open but is operationally heavy.

## Privacy domains

- **Stored data: none.** Indexed data is public chain state, served as-is.
- **Query privacy: absent.** The serving operator sees what a client queries; the leak described in the [Private Read use case](../use-cases/private-read.md).
- **Encrypted payload anchoring.** Client-side-encrypted records anchored on-chain are indexable as ciphertext, establishing existence, ordering, and completeness without plaintext access; keys stay with the owner.

## Enterprise demand and use cases

- **Audit and disclosure read path:** an institution keeps records private but publishes a tamper-evident fingerprint of each on-chain. Its auditor, shown a sample, must establish the sample is complete rather than curated. An indexed, independently re-derivable record of every published fingerprint lets the auditor check both directions: each disclosed record matches a published entry, and no published entry is missing from the disclosure.
- **Compliance monitoring substrate:** indexed views and evidence exports feeding rule engines and case management.
- **Market and risk monitoring** for institutions and supervisors needing a non-conflicted source.
- **Application backends:** the incumbent, high-volume use; regulated deployments typically buy through hosting operators.

## Technical details

- Deterministic end to end: content-addressed inputs and pinned code versions yield byte-identical re-runs.
- Reorg-aware; resumable cursors; historical queries at past blocks.
- Serving surfaces: GraphQL APIs, gRPC streams, SQL and file sinks.
- Licenses: Apache 2.0 (firehose-core, substreams, graph-node), MIT (indexer components), GPL-2.0 (protocol contracts).

## Strengths

- **Neutrality:** many independent operators; no affiliated exchange, analytics, or trading business.
- **Re-performability:** auditors re-run the open pipeline and compare hashes instead of trusting served output.
- **Accountable responses:** network responses are signed and disputable with staked collateral; conventional data API answers are deniable.
- **Coverage and maturity:** live since 2020; per-network status in the [networks registry](https://networks-registry.thegraph.com/).
- **Exit paths:** open licenses, public underlying data, self-hosting.

## Risks and open questions

- Query results are attested, not proof-carrying: extraction-level verification exists, but validity proofs for derived results are research.
- Completeness ceiling: an index attests on-chain emissions, never what went unrecorded nor off-chain reality; indexer output is not evidence of reserves or solvency.
- Read privacy: query patterns leak to the serving operator; multi-server private information retrieval over independent indexers is research.
- Serving plaintext is not on offer: The Graph serves ciphertext and the client decrypts, which works today and is an application concern. Indexers would need viewing keys only for general-purpose plaintext queries. Restricted predicates need no keys and largely work at the application layer today, since a deterministically encrypted field can be matched for equality as bytes; but such schemes buy selectivity by leaking structure (equality patterns, ordering), with a real inference-attack literature. General computation over ciphertext (FHE) is far from query-serving latency; enclave-based serving is research.
- Accountability is economic and procedural, not cryptographic: disputes are settled by a governance-appointed arbitrator, and protocol contracts are upgradeable under council governance.
- The decentralized network offers no SLA or SOC-type report; those come from hosting operators, which reintroduce single-operator trust.

## Links

- [https://thegraph.com](https://thegraph.com)
- [https://github.com/graphprotocol/graph-node](https://github.com/graphprotocol/graph-node)
- [https://github.com/streamingfast/firehose-core](https://github.com/streamingfast/firehose-core)
- [https://github.com/streamingfast/substreams](https://github.com/streamingfast/substreams)
- [https://github.com/graphprotocol/veemon](https://github.com/graphprotocol/veemon)

