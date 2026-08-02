# Changelog

All notable changes to the EthSystems Map are documented here.

## [Unreleased]

- feat(pattern): [Private Geospatial Attestation](patterns/pattern-private-geospatial-attestation.md) -- prove a plot polygon does not intersect a committed deforestation layer without revealing coordinates, with the reference-layer commitment named as the load-bearing assumption ([#184](https://github.com/ethsystems/map/pull/184))
- feat(approach|use-case): [Approach: Private Supply Chain](approaches/approach-private-supply-chain.md), replacing the "Approach TBD" stub in section 5 of [Private Supply Chain](use-cases/private-supply-chain.md) and adding EUDR to its regulatory drivers ([#184](https://github.com/ethsystems/map/pull/184))

## [0.4.0] - 2026-07-02

23 commits, 166 files changed since [v0.3.0](https://github.com/ethsystems/map/releases/tag/v0.3.0) (Apr 2026). Major additions: resilience use cases (civic participation, disbursement rails, identity continuity), I2U protection patterns, pattern/approach schema v2 (strict flip) with CROPS and post-quantum analysis, domain reframing beyond FIs, and a Q2 2026 content QA audit.

- chore: Q2 2026 content audit pass across all patterns, use cases, approaches, domains, jurisdictions, and vendors; corrections tracked in [QA-AUDIT.md](QA-AUDIT.md) ([#130](https://github.com/ethsystems/map/pull/130))
- chore: remove the dormant `weekly-updates/` directory and `scripts/weekly-summary.sh` (single entry, unused since 2026-01) ([#130](https://github.com/ethsystems/map/pull/130))
- docs(domains): reframe [Domains index](domains/README.md), [Payments](domains/payments.md), [Identity & Compliance](domains/identity-compliance.md), [Funds & Assets](domains/funds-assets.md), [Trading](domains/trading.md), and [Data & Oracles](domains/data-oracles.md) to cover public-sector, NGO, and resilience contexts alongside FIs; add [Civic Coordination & Governance](domains/governance.md) domain; add `governance` to the use-case schema enum ([#170](https://github.com/ethsystems/map/pull/170), closes [#169](https://github.com/ethsystems/map/issues/169))
- refactor(use-cases): rename `## 2) Additional Business Context` to `## 2) Additional Context` across [_template](use-cases/_template.md) and 23 use-case cards; strip reader-facing "Confidential context" placeholder line from 20 private-* cards ([#170](https://github.com/ethsystems/map/pull/170), closes [#169](https://github.com/ethsystems/map/issues/169))
- feat(use-case): [Resilient Civic Participation](use-cases/resilient-civic-participation.md) -- credentialed petition protocol with per-signer forward-secure ratcheting, blob-anchored signature batches, and a chain-state-only resolution SNARK (ECI, repository governance, workplace organising); adds [Approach: Civic Participation](approaches/approach-civic-participation.md) and generic patterns [Forward-Secure Pseudorandom Tree](patterns/pattern-forward-secure-pseudorandom-tree.md) and [Blob-Anchored State With KZG Dispute](patterns/pattern-blob-anchored-state-with-dispute.md) ([#165](https://github.com/ethsystems/map/pull/165))
- chore(approach): I2U analysis pass across [Private Payments](approaches/approach-private-payments.md), [White-Label Deployment](approaches/approach-white-label-deployment.md), and [Private Broadcasting](approaches/approach-private-broadcasting.md). Adds [Self-Custodial Deployment](approaches/approach-white-label-deployment.md) (context: i2u) to White-Label. Relabels L1 Shielded Payments, Stateless Plasma, and Private Rollups from `i2i` to `both` where the underlying technology already serves end users. Closes [`#124`](https://github.com/ethsystems/map/issues/124) ([`#167`](https://github.com/ethsystems/map/pull/167))
- feat(pattern): [Private Information Retrieval](patterns/pattern-private-information-retrieval.md), plugs client-side query leakage in shielded-pool path retrieval and note discovery, and applies to any indexed lookup whose index is itself sensitive ([`#168`](https://github.com/ethsystems/map/pull/168), companion to [ethsystems/pocs#68](https://github.com/ethsystems/pocs/issues/68))
- feat(pattern): [Immutable Guarantees](patterns/pattern-immutable-guarantees.md), separating mutable operational parameters from immutable safety invariants, with a published invariant set that survives hostile operators ([#166](https://github.com/ethsystems/map/pull/166), closes [#127](https://github.com/ethsystems/map/issues/127))
- fix(domains): refresh Adjacent vendors across [payments](domains/payments.md), [custody](domains/custody.md), [data-oracles](domains/data-oracles.md), [funds-assets](domains/funds-assets.md), [identity-compliance](domains/identity-compliance.md), and [trading](domains/trading.md). Remove Chainlink ACE (weak CROPS fit on openness and censorship resistance) and backfill with CROPS-aligned vendors already documented in [vendors/](vendors/) ([#164](https://github.com/ethsystems/map/pull/164)).
- fix(ci): stabilize [full markdown link checks](.github/workflows/link-check-full.yml) by ignoring bot-blocked external references and repairing stale internal links ([#163](https://github.com/ethsystems/map/pull/163))
- feat(schema): flip [pattern.json](scripts/schemas/pattern.json) and [validate-patterns.js](scripts/validate-patterns.js) to strict v2. All v1 aliases (`os`, `privacy`, `security` CROPS keys; `PoC`/`pilot`/`prod`/`experimental` maturity values; `privacy_goal`, `assumptions`, `dependencies`, `## Ingredients`, `## Guarantees` section names) now fail validation instead of warning. ([#156](https://github.com/ethsystems/map/pull/156))
- feat(schema): approach template v2 -- replace [_template.md](approaches/_template.md) with the v2 skeleton from [#151](https://github.com/ethsystems/map/issues/151) and update [validate-patterns.js](scripts/validate-patterns.js) to check v2 sections (`## Problem framing`, `## Approaches`, `## Comparison`, `## Persona perspectives`, `## Recommendation`, `## Open questions`), required frontmatter (`title`, `status`, `last_reviewed`, `use_case`, `primary_patterns`), and the `^Approach:\s` title prefix. Lenient mode (warnings only) until a future strict-flip ([#161](https://github.com/ethsystems/map/pull/161))
- refactor(approach): port 9 approach cards in [approaches/](approaches/) to v2 schema (frontmatter, per-sub-approach YAML blocks, comparison table, persona perspectives, recommendation) ([#160](https://github.com/ethsystems/map/pull/160), [#151](https://github.com/ethsystems/map/issues/151))
- chore(approach): remove `approaches/approach-privacy-standards-survey.md` (does not fit v2 schema; relocation tracked in [#159](https://github.com/ethsystems/map/issues/159)) ([#160](https://github.com/ethsystems/map/pull/160))
- feat(use-case): [Resilient Disbursement Rails](use-cases/resilient-disbursement-rails.md) -- humanitarian disbursement to recipients in adversarial jurisdictions, with off-ramp unlinkability as the primary cryptographic requirement ([#157](https://github.com/ethsystems/map/pull/157))
- feat(pattern): [Forward-Secure Signatures](patterns/pattern-forward-secure-signatures.md), [Mesh Store-and-Forward Submission](patterns/pattern-mesh-store-forward-submission.md), [Relay-Mediated Proving](patterns/pattern-relay-mediated-proving.md), and [Recipient-Derived Receive Addresses](patterns/pattern-recipient-derived-receive-addresses.md) -- generic patterns supporting constrained-signer participation in proof-gated systems ([#157](https://github.com/ethsystems/map/pull/157))
- feat(approach): Resilient Disbursement Rails section in [Private Payments](approaches/approach-private-payments.md) for humanitarian disbursement under adversarial-jurisdiction threat models ([#157](https://github.com/ethsystems/map/pull/157))
- feat(schema): pattern template v2 -- structured `crops_context`, `post_quantum`, `context_differentiation`, `related_patterns`, `open_source_implementations`, `sub_patterns`, and `type: meta` flag in [_template.md](patterns/_template.md), [pattern.json](scripts/schemas/pattern.json), and [validate-patterns.js](scripts/validate-patterns.js). Validator stays permissive during migration; existing v1 patterns continue to pass with deprecation warnings. ([#152](https://github.com/ethsystems/map/pull/152), [#150](https://github.com/ethsystems/map/issues/150))
- feat(pattern): [ZK Wrappers](patterns/pattern-zk-wrappers.md) -- verify off-chain signatures (passport, DKIM, attestation) inside a ZK circuit for attribute proofs without credential disclosure ([#153](https://github.com/ethsystems/map/pull/153))
- feat(pattern): [Social Recovery](patterns/pattern-social-recovery.md) -- guardian-quorum rotation of account keys or identity anchors without issuer or custodian involvement ([#153](https://github.com/ethsystems/map/pull/153))
- feat(pattern): [Onion Routing](patterns/pattern-onion-routing.md) -- Tor-based multi-hop relay for sender IP anonymity (PSE tor-js, Flashbots .onion) ([#146](https://github.com/ethsystems/map/pull/146))
- feat(pattern): [Mixnet Anonymity](patterns/pattern-mixnet-anonymity.md) -- batching, reordering, and cover traffic for strongest network anonymity (Nym, HOPR) ([#146](https://github.com/ethsystems/map/pull/146))
- refactor(pattern): split [Network-Level Anonymity](patterns/pattern-network-anonymity.md) into umbrella pattern linking onion routing, mixnet, and TEE sub-patterns ([#146](https://github.com/ethsystems/map/pull/146))
- feat(pattern): [User-Controlled Viewing Keys](patterns/pattern-user-controlled-viewing-keys.md) -- user-held viewing key custody for I2U privacy sovereignty ([#146](https://github.com/ethsystems/map/pull/146))
- feat(pattern): enhance [Network-Level Anonymity](patterns/pattern-network-anonymity.md) with I2U analysis ([#146](https://github.com/ethsystems/map/pull/146))
- feat(pattern): [zk-promises](patterns/pattern-zk-promises.md) -- stateful anonymous credentials with async callbacks for blind compliance enforcement ([#132](https://github.com/ethsystems/map/pull/132))
- feat(pattern): [Proof of Innocence](patterns/pattern-proof-of-innocence.md) -- association set membership/exclusion proofs for compliance without surveillance ([#132](https://github.com/ethsystems/map/pull/132))
- feat(approach|use-case): [Resilient Identity Continuity](use-cases/resilient-identity-continuity.md) and [added approach](approaches/approach-private-identity.md) ([#145](https://github.com/ethsystems/map/pull/145))

### Removed

- chore: Retire `PRD-IPTF-PUBLIC-Q1-2026.md` -- Q1 closed; remaining unfinished items tracked in [#143](https://github.com/ethsystems/map/issues/143) ([#144](https://github.com/ethsystems/map/pull/144))

## [0.3.0] - 2026-04-13

71 commits, 162 files changed since v0.2.0. Major additions: CROPS evaluation framework, 20+ new patterns, 12 new use cases, 4 new approaches, post-quantum threat analysis, CI quality guardrails.

### Added

#### Patterns
- feat(pattern): [Forced Withdrawal](patterns/pattern-forced-withdrawal.md) — L1 escape hatch for asset recovery without operator cooperation ([#125](https://github.com/ethsystems/map/issues/125))
- feat(pattern): [TLS Payment Bridge](patterns/pattern-tls-payment-bridge.md) — trust-minimized fiat-to-onchain swaps via zk-TLS proofs on instant payment rails ([#88](https://github.com/ethsystems/map/issues/88))
- feat(pattern): [Native Account Abstraction](patterns/pattern-native-account-abstraction.md) — EIP-8141 modular verification logic for PQ auth agility ([#121](https://github.com/ethsystems/map/pull/121))
- feat(pattern): [ZK Proof Systems](patterns/pattern-zk-proof-systems.md) — taxonomy of proving systems with PQ safety analysis ([#121](https://github.com/ethsystems/map/pull/121))
- feat(pattern): [Private Set Intersection (DH-based)](patterns/pattern-private-set-intersection-dh.md) - Bilateral ECDH-PSI for private matching of identifier sets
- feat(pattern): [Private Set Intersection (OPRF-based)](patterns/pattern-private-set-intersection-oprf.md) - OT/OPRF-based PSI scaling to millions of elements
- feat(pattern): [Private Set Intersection (Circuit-based)](patterns/pattern-private-set-intersection-circuit.md) - Garbled circuit PSI for computing arbitrary functions over intersections
- feat(pattern): [Private Set Intersection (FHE-based)](patterns/pattern-private-set-intersection-fhe.md) - FHE-based PSI for asymmetric set sizes with post-quantum security
- feat(pattern): [Permissionless Spend Auth](patterns/pattern-permissionless-spend-auth.md) - Outer/inner circuit split for permissionless, user-chosen spend authorization without fragmenting anonymity sets (EIP-8182)
- feat(pattern): [Safe Proof Delegation](patterns/pattern-safe-proof-delegation.md) - Intent-based delegation to non-custodial provers with revocable visibility (EIP-8182)
- feat(pattern): [Network-Level Anonymity](patterns/pattern-network-anonymity.md) - Umbrella pattern for transport-layer sender anonymity (Tor, mixnets, private RPC, TEE-assisted, VPN)
- feat(pattern): [TEE-Assisted Network Anonymity](patterns/pattern-tee-network-anonymity.md) - TEE+secret-sharing approach for low-latency sender anonymity (Flashbots Flashnet)
- feat(pattern): Private Shared State split into [co-SNARKs](patterns/pattern-private-shared-state-cosnark.md), [FHE](patterns/pattern-private-shared-state-fhe.md), [TEE](patterns/pattern-private-shared-state-tee.md) — each with distinct CROPS profile and trust model ([#104](https://github.com/ethsystems/map/issues/104))
- feat(pattern): Enhanced [Hybrid TEE + ZK Settlement](patterns/pattern-tee-zk-settlement.md) with trust framework, TEE API surface, stealth address design, anti-pattern table, and PoC learnings ([#79](https://github.com/ethsystems/map/issues/79))
- feat(pattern): [Compliance Monitoring](patterns/pattern-compliance-monitoring.md) - Transaction screening with privacy-preserving audit trails ([#73](https://github.com/ethsystems/map/pull/73))
- feat(pattern): [L2 Privacy Evaluation Framework](patterns/pattern-l2-privacy-evaluation.md) - Methodology for institutions to compare privacy L2s (PR-011)
- feat(pattern): [Cross-chain Privacy Bridge](patterns/pattern-cross-chain-privacy-bridge.md) - Bridge assets between chains while preserving privacy
- feat(pattern): [Stateless Plasma Privacy](patterns/pattern-plasma-stateless-privacy.md) - Client-side proving with minimal on-chain footprint (Intmax-style)
- feat(pattern): [vOPRF Nullifiers](patterns/pattern-voprf-nullifiers.md) - Threshold vOPRF-based nullifier generation for credentials/signals ([#61](https://github.com/ethsystems/map/pull/61))
- feat(pattern): [Modular Privacy Stack](patterns/pattern-modular-privacy-stack.md) - Four-layer privacy architecture ([#54](https://github.com/ethsystems/map/pull/54))
- feat(pattern): [Hybrid public-private modes](patterns/pattern-hybrid-public-private-modes.md)
- feat(pattern): [Private transaction broadcasting](patterns/pattern-private-transaction-broadcasting.md) ([#43](https://github.com/ethsystems/map/pull/43))
- feat(pattern): [TEE-based privacy](patterns/pattern-tee-based-privacy.md) ([#44](https://github.com/ethsystems/map/pull/44))
- feat(pattern): [Threshold encrypted mempool](patterns/pattern-threshold-encrypted-mempool.md) ([#45](https://github.com/ethsystems/map/pull/45))

#### Approaches
- feat(approach): [Private Money Market Funds](approaches/approach-private-money-market-funds.md) - Privacy-preserving MMF operations with ZK NAV proofs
- feat(approach): Restructured [Private Trade Settlement](approaches/approach-private-trade-settlement.md) — separated single-chain and cross-chain approaches, added TEE+ZK, MPC, and intent-based settlement with trade-off matrices ([#77](https://github.com/ethsystems/map/issues/77))
- feat(approach): Enhanced [Private Bonds](approaches/approach-private-bonds.md) with PoC learnings, coprocessor model analysis (co-SNARKs vs FHE), comparison matrix, and implementation guidance
- feat(approach): [Privacy Standards Survey](approaches/approach-privacy-standards-survey.md) - Standards catalog, gap analysis, and decision guidance ([#64](https://github.com/ethsystems/map/pull/64))
- feat(approach): [White-label infrastructure deployment](approaches/approach-white-label-deployment.md) ([#55](https://github.com/ethsystems/map/pull/55))
- feat(approach): [Atomic DvP Settlement](approaches/approach-dvp-atomic-settlement.md) ([#56](https://github.com/ethsystems/map/pull/56))
- feat(approach): Expanded [Private Payments](approaches/approach-private-payments.md) with Plasma and TEE approaches

#### Use Cases
- feat(use-case): [Private Supply Chain](use-cases/private-supply-chain.md), [Private Procurement](use-cases/private-procurement.md), [Private Registry](use-cases/private-registry.md), [Private Read](use-cases/private-read.md), [Private Corporate Bonds](use-cases/private-corporate-bonds.md), [Private Government Debt](use-cases/private-government-debt.md), [Private FX](use-cases/private-fx.md), [Private Stocks](use-cases/private-stocks.md), [Private Commodities](use-cases/private-commodities.md), [Private Repo](use-cases/private-repo.md), [Private Money Market Funds](use-cases/private-money-market-funds.md), [Private Treasuries](use-cases/private-treasuries.md), [Private Payments](use-cases/private-payments.md), [Private Oracles](use-cases/private-oracles.md), [Private Messaging](use-cases/private-messaging.md)

#### Domains & Vendors
- feat(domain): [Post-Quantum Threats](domains/post-quantum.md) — PQ threat landscape, Ethereum layer analysis, and application-layer breakage index ([#121](https://github.com/ethsystems/map/pull/121))
- feat(vendor): [Peer](vendors/peer.md) — P2P fiat-to-crypto onramp using TLSNotary proofs ([#88](https://github.com/ethsystems/map/issues/88))
- feat(vendor): [EY Starlight](vendors/ey.md#starlight) - Solidity transpiler for private on-chain state
- feat(vendor): [TACEO Merces](vendors/taceo-merces.md) - MPC + ZK approach for private stablecoin transfers
- feat(vendor): [Fhenix](vendors/fhenix.md) - FHE privacy ([#32](https://github.com/ethsystems/map/pull/32))
- feat(vendor): [Space and Time](vendors/space-and-time.md) ([#46](https://github.com/ethsystems/map/pull/46))

#### CROPS Framework & Evaluation
- feat(template): [crops_profile](patterns/_template.md) frontmatter block added to pattern template for CROPS dimension indexing ([#101](https://github.com/ethsystems/map/issues/101))
- feat(template): [crops_profile](vendors/_template.md) frontmatter block and CROPS evaluation criteria added to vendor template and README ([#102](https://github.com/ethsystems/map/issues/102))
- docs(glossary): [CROPS, I2I, I2U](GLOSSARY.md#ethereum-systems-evaluation-frameworks) evaluation framework definitions ([#100](https://github.com/ethsystems/map/issues/100))
- docs(readme): [Evaluation Framework](README.md#evaluation-framework) section with CROPS alignment statement ([#100](https://github.com/ethsystems/map/issues/100))
- feat(patterns): CROPS-aligned 18 patterns with `crops_profile` frontmatter and deployment-context Trade-offs notes ([#104](https://github.com/ethsystems/map/issues/104))
- feat(patterns): Add `context` and `crops_profile` CROPS frontmatter to additional patterns

#### CI & Quality
- feat(ci): Enhanced AI content quality guardrails ([#58](https://github.com/ethsystems/map/issues/58))
  - Vale prose linter with custom EthSystems styles for marketing language, hedging, and terminology
  - GLOSSARY.md term consistency checker (`scripts/check-terminology.js`)
  - Extended validation to all content types (vendors, use-cases, approaches, jurisdictions)
  - JSON Schema validation for frontmatter
  - Husky pre-commit hooks with lint-staged
  - LLM-based content review tool (`scripts/llm-review.js`)
- feat(ci): Pattern validation workflow ([#40](https://github.com/ethsystems/map/pull/40))

#### Documentation
- docs(glossary): Add PQ cryptography terms (CRQC, HNDL, ML-KEM, ML-DSA, SLH-DSA, Poseidon) ([#121](https://github.com/ethsystems/map/pull/121))
- docs: Q1 2026 PRD with sprint planning ([#39](https://github.com/ethsystems/map/pull/39))
- docs: [CHANGELOG.md](CHANGELOG.md) and weekly summary script ([#49](https://github.com/ethsystems/map/pull/49))
- docs: [weekly-updates](weekly-updates/) directory ([#59](https://github.com/ethsystems/map/pull/59))

### Changed

- chore(approach|use-case): Updated [Private Payments](approaches/approach-private-payments.md) approach and [use case](use-cases/private-payments.md) with PoC validation data
- refactor(use-case): Renamed Private Authentication → [Private Identity](use-cases/private-identity.md) — authentication reframed as one application of private identity (also renamed approach)
- feat(use-case): Enriched [Private Payments](use-cases/private-payments.md) with conditional & programmable payment privacy (grant disbursement, milestone-based releases)
- feat(use-case): Enriched [Private Government Debt](use-cases/private-government-debt.md) with fiscal transparency vs operational privacy tension
- feat(use-case): Enriched [Private Identity](use-cases/private-identity.md) with credential portability & reuse problem, government credential sources
- feat(use-case): Enriched [Private Oracles](use-cases/private-oracles.md) with data provenance & consent angle
- feat(use-case): Broadened [Private Identity](use-cases/private-identity.md) beyond institutional KYC to cover governance and national identity use cases
- feat(approach): Restructured [Private Identity Approach](approaches/approach-private-identity.md) with credential-source taxonomy and validated deployment references
- fix(patterns): Add PQ exposure notes to Trade-offs of 16 affected patterns and cross-refs to 4 PQ-aware patterns ([#121](https://github.com/ethsystems/map/pull/121))
- fix(pattern): Refreshed [TEE-Based Privacy](patterns/pattern-tee-based-privacy.md) — added CPU-encrypted vs hypervisor-isolated platform classification and threat model comparison

### Removed

- refactor(patterns): Deleted 4 redundant patterns ([#104](https://github.com/ethsystems/map/issues/104)):
  `pattern-zk-derivative-delta.md` (covered by shielding + co-SNARKs),
  `pattern-zk-htlc.md` (covered by cross-chain privacy bridge + shielding),
  `pattern-zk-shielded-balances.md` (merged into [Shielding](patterns/pattern-shielding.md)),
  `pattern-zk-spv.md` (removed from map)
- refactor(patterns): Deleted original `pattern-private-shared-state.md`, replaced by 3 trust-model-specific cards ([#104](https://github.com/ethsystems/map/issues/104))

### Fixed

- fix(pattern): include tradeoff around handling of stock splits in [ERC-3643](patterns/pattern-erc3643-rwa.md) ([#81](https://github.com/ethsystems/map/pull/81))
- fix(refs): Updated [Private Bonds](use-cases/private-bonds.md) PoC links to ethsystems/pocs
- fix(pattern): Required frontmatter fields across all patterns ([#42](https://github.com/ethsystems/map/pull/42))

## [0.2.0] - 2025-12-19 (End of Year)

### Added

- feat(pattern): [TEE key manager](patterns/pattern-tee-key-manager.md) ([#33](https://github.com/ethsystems/map/pull/33))
- feat(pattern): [EIL](patterns/pattern-eil.md) - Encrypted Inline Ledger ([#26](https://github.com/ethsystems/map/pull/26))
- feat(pattern): [FOCIL-EIP7805](patterns/pattern-focil-eip7805.md) ([#26](https://github.com/ethsystems/map/pull/26))
- feat(pattern): [Lean Ethereum](patterns/pattern-lean-ethereum.md) ([#26](https://github.com/ethsystems/map/pull/26))
- feat(pattern): [OIF](patterns/pattern-oif.md) - Optimized Integrity Framework ([#26](https://github.com/ethsystems/map/pull/26))
- feat(pattern): [Noir private contracts](patterns/pattern-noir-private-contracts.md) ([#21](https://github.com/ethsystems/map/pull/21))
- feat(vendor): [Paladin](vendors/paladin.md) ([#19](https://github.com/ethsystems/map/pull/19))
- feat(vendor): [State Labs](vendors/tx-shield.md) - Tx Shield, OpenTMP LLM, Collab-Key ([#7](https://github.com/ethsystems/map/pull/7))
- feat(vendor): [Soda Labs](vendors/soda-labs.md)
- feat(vendor): [Miden](vendors/miden.md) docs ([#18](https://github.com/ethsystems/map/pull/18))
- feat(approach): [Private broadcasting](approaches/approach-private-broadcasting.md) ([#6](https://github.com/ethsystems/map/pull/6))
- feat(approach): [Private bonds](approaches/approach-private-bonds.md)
- feat(jurisdiction): [EU Data Protection](jurisdictions/eu-data-protection.md) ([#8](https://github.com/ethsystems/map/pull/8))
- docs: [GLOSSARY.md](GLOSSARY.md) - privacy terminology ([#5](https://github.com/ethsystems/map/pull/5))

### Changed

- refactor: Split patterns directory into patterns + approaches ([#2](https://github.com/ethsystems/map/pull/2))

### Fixed

- fix(pattern): [DvP ERC-7573](patterns/pattern-dvp-erc7573.md) updates ([#31](https://github.com/ethsystems/map/pull/31))
- fix(use-case): [Private Identity](use-cases/private-identity.md) - revocation, zk-TLS mechanism
- fix(docs): Glossary - clarified core privacy concepts ([#29](https://github.com/ethsystems/map/pull/29))
- fix(docs): ZKsync naming standardization ([#20](https://github.com/ethsystems/map/pull/20))

## [0.1.0] - 2025-10-06 (MVP)

### Added

- feat: Initial repository import with 62 files
- feat(pattern): 20+ privacy patterns (ZK, MPC, TEE, stealth addresses, etc.)
- feat(use-case): [private-identity](use-cases/private-identity.md)
- feat(use-case): [private-bonds](use-cases/private-bonds.md)
- feat(use-case): [private-derivatives](use-cases/private-derivatives.md)
- feat(use-case): [private-rwa-tokenization](use-cases/private-rwa-tokenization.md)
- feat(use-case): [private-stablecoins](use-cases/private-stablecoins.md)
- feat(domain): [custody](domains/custody.md)
- feat(domain): [data-oracles](domains/data-oracles.md)
- feat(domain): [funds-assets](domains/funds-assets.md)
- feat(domain): [identity-compliance](domains/identity-compliance.md)
- feat(domain): [payments](domains/payments.md)
- feat(domain): [trading](domains/trading.md)
- feat(jurisdiction): [DE eWpG](jurisdictions/de-eWpG.md)
- feat(jurisdiction): [EU MiCA](jurisdictions/eu-MiCA.md)
- feat(jurisdiction): [Banking Secrecy](jurisdictions/int-banking-secrecy.md)
- feat(jurisdiction): [US SEC](jurisdictions/us-SEC.md)
- feat(vendor): [Aztec](vendors/aztec.md)
- feat(vendor): [Chainlink ACE](vendors/chainlink-ace.md)
- feat(vendor): [Curvy](vendors/curvy.md)
- feat(vendor): [Fireblocks](vendors/fireblocks.md)
- feat(vendor): [Miden](vendors/miden.md)
- feat(vendor): [PrivacyPools](vendors/privacypools.md)
- feat(vendor): [Railgun](vendors/railgun.md)
- feat(vendor): [Renegade](vendors/renegade.md)
- docs: [GLOSSARY.md](GLOSSARY.md) with standardized terminology
- docs: CC0 public domain license

[Unreleased]: https://github.com/ethsystems/map/compare/v0.3.0...HEAD
[0.3.0]: https://github.com/ethsystems/map/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/ethsystems/map/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/ethsystems/map/releases/tag/v0.1.0
