---
title: "Pattern: Reproducible Audit Extraction"
status: draft
maturity: testnet
type: standard
layer: offchain
last_reviewed: 2026-07-16

works-best-when:
  - An auditor, regulator, or counterparty must verify that disclosed or reported figures reflect the complete set of on-chain emissions, not a curated subset.
  - Disclosure patterns (viewing keys, predicate proofs, encrypted audit logs) are in place and the open question is what the disclosed material is checked against.
  - The verifying party can run, or commission, an independent re-execution of the data pipeline.
avoid-when:
  - The facts to verify are off-chain (bank balances, custodial holdings); no extraction of chain data can establish them.
  - The verifier will never re-execute; without at least sampled re-runs the pattern collapses to trusting the pipeline operator.

context: both
context_differentiation:
  i2i: "An institution's auditor or a supervisory authority re-runs the published pipeline over public inputs and compares output hashes, replacing reliance on the reporting institution's (or its data vendor's) pipeline with reliance on canonical chain data. Bilateral disputes reduce to a hash mismatch that either party can demonstrate."
  i2u: "End users and the public can verify institutional claims anchored on-chain (supply figures, registry roots, log anchors) without permission from the institution, provided the recipe is published. The binding constraint is resources: re-execution favors parties who can run infrastructure, so public-interest verification in practice depends on independent third parties publishing their re-runs."

crops_profile:
  cr: high
  o: yes
  p: none
  s: medium

crops_context:
  cr: "Verification is permissionless: inputs are public chain data and the recipe is published, so no operator can prevent an independent party from re-deriving the result. Drops to `medium` when the canonical input view itself depends on a small set of archive-data providers for deep history."
  o: "Open by construction: the pattern does not work with a closed recipe, since the verifier must re-run it. Several implementations under permissive and copyleft licenses exist. Deployments that keep transform code proprietary are not instances of this pattern."
  p: "None. The pattern operates on public emissions (events, commitments, nullifiers, anchors, attestation logs) and adds no confidentiality. Privacy of the underlying business data comes from the shielding and disclosure patterns it composes with; this card covers the verification leg those patterns presuppose."
  s: "Rests on determinism engineering (pinned toolchains, content-addressed inputs, no wall-clock or float nondeterminism) and on the verifier obtaining a canonical view of the chain. No consensus, committee, or hardware trust is introduced; the residual risk is nondeterminism bugs and verifiers who never actually re-run."

post_quantum:
  risk: low
  vector: "The pattern itself relies on hash functions (content addressing, Merkle anchors), which retain security against a CRQC up to Grover-type reductions. Signature and commitment exposure is inherited from the host chain and from the composed disclosure patterns, not added here."
  mitigation: "Use hash constructions with adequate post-quantum margins for content addressing and anchors (SHA-2/SHA-3, Poseidon per its stated margins). See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: []

related_patterns:
  composes_with: [pattern-l2-encrypted-offchain-audit, pattern-regulatory-disclosure-keys-proofs, pattern-compliance-monitoring, pattern-commit-and-prove]
  see_also: [pattern-verifiable-attestation, pattern-private-information-retrieval]

open_source_implementations:
  - url: https://github.com/streamingfast/firehose-core
    description: "Firehose: deterministic, content-addressed flat-file extraction of chain history (The Graph / StreamingFast, Apache 2.0)"
    language: "Go"
  - url: https://github.com/streamingfast/substreams
    description: "Substreams: deterministic WASM transform modules over Firehose files (Apache 2.0)"
    language: "Rust"
  - url: https://github.com/TrueBlocks/trueblocks-core
    description: "TrueBlocks: local, reproducible index of Ethereum address appearances (GPL-3.0)"
    language: "Go"
  - url: https://github.com/paradigmxyz/cryo
    description: "cryo: deterministic extraction of chain data to Parquet/CSV datasets (Apache 2.0)"
    language: "Rust"
  - url: https://github.com/graphprotocol/graph-node
    description: "graph-node: deterministic indexing of extracted data into a queryable entity store, with proofs of indexing over the result (Apache 2.0)"
    language: "Rust"
  - url: https://github.com/subsquid/squid-sdk
    description: "Subsquid: independent indexing framework serving queryable indexes over extracted chain data (Apache 2.0)"
    language: "TypeScript"
---

## Intent

Give auditors, regulators, and counterparties a read path over public chain state they can independently re-execute to byte-identical output. Disclosure patterns govern who sees plaintext; this pattern establishes what the complete emitted record was, so a scoped disclosure can be checked against it.

## Components

- Deterministic extraction: canonical blocks, receipts, and logs written to content-addressed files traceable to block hashes.
- Deterministic transform: versioned code (pinned dependencies, no nondeterministic ordering, no wall-clock or floating-point dependence) deriving the report-shaped output (a supply figure, a registry root).
- Re-execution manifest: input content hashes, transform code version and hash, output hashes, and the block range covered.
- Completeness scope: the contract addresses, event signatures, and block range that bound "the complete set" for an engagement.
- Optional on-chain anchor: a hash or Merkle root of the output posted on-chain as a fixed point for later verification.
- Serving layer: a queryable index over the extracted record, so a relying party can ask completeness-shaped questions (every in-scope entry across a range, in order, with gaps and conflicts surfaced) without re-running the pipeline. It is audit infrastructure rather than convenience only if it **reports what the record contains**, including malformed and conflicting entries, since dropping them silently is indistinguishable from their absence, and if the serving party is **identifiable and answerable** for a wrong answer. Serving never replaces re-execution as the trust mechanism.

## Protocol

1. [operator] Extract the in-scope chain data into content-addressed files anchored to block hashes.
2. [operator] Run the versioned transform; produce outputs plus the re-execution manifest.
3. [operator] Deliver outputs and manifest to the relying party; optionally anchor the output hash.
4. [auditor] Obtain a canonical chain view independently (own node, or light-client-verified headers) and re-run extraction and transform from the manifest.
5. [auditor] Compare output hashes; byte-identical output verifies the pipeline, a mismatch localizes the dispute to specific inputs or code versions.
6. [regulator] Check that material disclosed under a viewing key or proof reconciles against the re-derived record, both directions: every anchored commitment accounted for, none added, none omitted.

## Guarantees & threat model

Guarantees:

- Faithfulness: any party can re-derive the figures from canonical chain data; the operator cannot alter inputs or logic without breaking hash equality.
- Completeness within scope: the re-derived record contains every in-scope emission, so a disclosure checked against it cannot omit anchored records.

Threat model:

- Completeness is relative to on-chain emissions. Facts never emitted (an unrecorded liability, an off-chain balance) are invisible to any extraction; closing that gap needs legal attestation, not this pattern.
- Nondeterminism silently breaks verification; determinism must be asserted in CI, not assumed.
- The verifier's chain view is a dependency: fed a wrong canonical view, it verifies the wrong record.
- Scope-definition gaming: an operator who controls the scope (addresses, event set) can exclude inconvenient contracts, so the scope must be fixed by the relying party or disclosed for challenge.
- A verifier who never re-runs receives no guarantee; sampled re-execution is the minimum honest posture.
- A serving layer that is not itself checkable becomes a new trusted party: where practical access is a served index rather than the verifier's own re-run, the guarantee degrades to trusting the server unless served output is periodically reconciled against re-derived output.

## Trade-offs

- The verifier does the work, and the cost depends on what the transform reads. Event-derived values are re-derivable from any node retaining historical receipts; values derived from contract-call results or execution traces need the EVM re-executed against archival state or a trace-capable node, a materially heavier requirement. Deep history is a third constraint: as history expiry advances, old ranges depend on independent archives, not on any node's retention policy. Proof-carrying alternatives (SNARK-verified SQL over committed tables) shift cost from verifier to prover but cover a narrower class of transforms and add cryptographic assumptions.
- Determinism engineering is real maintenance overhead across toolchain upgrades.
- Monthly or quarterly attestation cadences absorb re-run latency; real-time supervisory queries do not.

## Example

- A bond issuer settles on an L2, anchoring hourly Merkle roots of its encrypted trade log on-chain ([Low-cost L2 + Off-chain Encrypted Audit Log](pattern-l2-encrypted-offchain-audit.md)).
- At period close, the auditor receives the manifest, re-extracts the anchor events from an independent node, and re-derives the root chain byte-identically.
- A scoped viewing key on specific trades then confirms the disclosed records reconcile against the complete anchored set: nothing omitted, nothing inserted.
- A supervisor repeats the re-run a year later from the on-chain anchors alone, without the issuer's cooperation.

## See also

- [EIP-4444 (history expiry: why independent archives of chain history matter for deep re-runs)](https://eips.ethereum.org/EIPS/eip-4444)
- [Firehose documentation](https://firehose.streamingfast.io/)
- [TrueBlocks documentation](https://trueblocks.io/)
