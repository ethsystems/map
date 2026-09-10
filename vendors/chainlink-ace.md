---
title: "Vendor: Chainlink ACE"
status: ready
---

# Chainlink - ACE (Automated compliance for digital assets)

## What it is
Modular compliance layer built on the Chainlink Runtime Environment (CRE) to define, execute, and enforce identity- and policy-based controls for digital assets across public and private chains. Core onchain components (CCID, CCT Compliance Extension) pair with CRE services (Policy/Identity/Monitoring managers). Offchain policy execution returns signed approvals; onchain enforcement is required. PII/NPI remains offchain. It enables private transactions, with one of the main target use cases being CBDCs management.

## Fits with patterns
- [Regulatory Disclosure Keys Proofs](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Crypto Registry Bridge eWpG EAS](../patterns/pattern-crypto-registry-bridge-ewpg-eas.md)
- [DvP ERC-7573](../patterns/pattern-dvp-erc7573.md)

## Not a substitute for
- Privacy L2 or App-chain
- Identity and Attestations

## Architecture
- CCID (Cross-Chain Identity): onchain proofs of verified credentials (e.g., KYC/AML, accreditation) with offchain PII; compatible with LEI/vLEI, ONCHAINID, DIDs.
- CCT Compliance Extension: lightweight token interface that connects tokens to CCID, Policy Manager, and CCIP; applies to ERC-20, ERC-3643, and custom tokens.
- Policy Manager: policies defined offchain; executed onchain and/or offchain; always enforced onchain via contract hooks/modifiers at asset or protocol level.
- Identity Manager: middleware to register/sync credentials across networks without storing PII onchain; bridges real-world identity systems to onchain formats.
- Monitoring & Reporting Manager: alerts, audit logs, and evidence export; supports SOC integrations and automated pause/deny reactions.
- Cross-chain interop: CCIP used so identity/policy artifacts apply across networks; optional CCIP Private Transactions and Blockchain Privacy Manager for data minimization.
- Developer tooling: SDK, API, CLI, admin UI; compliance sandbox and audited onchain policy templates (allow/deny lists, RBAC, limits).

## Privacy domains
- Onchain policies
  - Fully verifiable in contracts; inputs/logic public.
- Offchain policies
  - Executed by oracle networks on CRE; return signed approvals; keep business rules private; enforcement happens onchain.
    - Open question: are policies private against operators of the network?
- Cross-chain private movement
  - CCIP Private Transactions + Blockchain Privacy Manager minimize onchain exposure when bridging private/public environments.

## Enterprise demand and use cases
- Segments: banks, custodians, fund admins, stablecoin issuers, regulated DeFi venues.
- Common flows: whitelist/eligibility checks, sanctions/deny lists, jurisdiction gating, fund subscription DvP, cross-chain settlement with policy gates, reusable onboarding (vLEI/ONCHAINID).
- Buyer profile: digital asset leads, compliance/risk teams seeking reusable identity and policy enforcement primitives across chains.

## Technical details
- Identity/credentials: onchain cryptographic proofs (CCID); offchain issuers/IDVs/regulators; lifecycle includes revocation/sync; optional EAS/attestation alignment.
- Token surface: CCT attaches policy/identity to tokens without rewriting token logic; pairs with CCIP for cross-chain.
- Policy execution model: define (offchain) -> execute (onchain and/or offchain) -> enforce (onchain) with evidence artifacts and signing key management.
- Interop/failure handling: CCIP provides routing/retries; policy enforcement guards settlement; monitoring emits anomalies and supports circuit breakers.
- Reporting: exportable logs/evidence for audits; institution controls disclosure scope.

## Strengths
- Hybrid model: private offchain decisions with mandatory onchain enforcement.
- Reusable identity and policy primitives that work across chains and token standards.
- Clear operational surfaces (monitoring/audit) aligned to regulated workflows.
- Developer velocity via templates, SDKs, and a sandbox.

## Risks and open questions
- Offchain execution path introduces operator trust unless coupled with strong attestations/TEEs; confirm evidence formats and SLAs.
- Identity governance and revocation across chains require clear ownership and DR plans.
- Cross-chain policy enforcement latency/failure modes need benchmarks.
- Privacy roadmap specifics (e.g., DECO vs TEEs adoption) vary by deployment; confirm per project.

## Links
- ACE product page: https://chain.link/automated-compliance-engine
- ACE technical overview: https://blog.chain.link/automated-compliance-engine-technical-overview/
- CRE overview: https://chain.link/cre
- CCIP overview: https://chain.link/cross-chain
- CCIP Private Transactions / Blockchain Privacy Manager: https://blog.chain.link/ccip-private-transactions-blockchain-privacy-manager/
- Chainlink Privacy Standard (privacy oracles/CRE): https://docs.chain.link/oracle-platform/privacy-standard