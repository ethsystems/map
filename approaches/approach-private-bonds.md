---
title: "Approach: Private Bond Issuance & Trading"
status: ready
last_reviewed: 2026-06-24

use_case: private-bonds
related_use_cases: [private-corporate-bonds, private-government-debt]

primary_patterns:
  - pattern-shielding
  - pattern-privacy-l2s
  - pattern-co-snark
  - pattern-icma-bdt-data-model
supporting_patterns:
  - pattern-private-stablecoin-shielded-payments
  - pattern-private-shared-state-fhe
  - pattern-user-controlled-viewing-keys
  - pattern-regulatory-disclosure-keys-proofs
  - pattern-verifiable-attestation
  - pattern-proof-of-innocence

pocs:
  folder: pocs/private-bond
  requirements: pocs/private-bond/REQUIREMENTS.md
  pocs:
    - name: "Custom UTXO"
      sub_approach: "UTXO Shielded Notes"
      spec: pocs/private-bond/custom-utxo/SPEC.md
      status: benchmarked
    - name: "Privacy L2 (Aztec)"
      sub_approach: "Privacy L2"
      spec: pocs/private-bond/privacy-l2/SPEC.md
      status: implemented
    - name: "FHE (Zama)"
      sub_approach: "FHE Coprocessor"
      spec: pocs/private-bond/fhe/SPEC.md
      status: implemented

open_source_implementations:
  - url: https://github.com/Railgun-Privacy/contract
    description: "Railgun (UTXO shielded pool, production volume)"
    language: Solidity
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec privacy-native L2"
    language: TypeScript / Noir
  - url: https://github.com/0xMiden/miden-base
    description: "Miden client-side ZK rollup"
    language: Rust
---

# Approach: Private Bond Issuance & Trading

## Problem framing

### Scenario

A bank issues a EUR 100M corporate bond series with private allocation amounts to 50 institutional investors and operates an active secondary market with RFQ-based price discovery. The bank needs hidden positions and trade sizes, atomic same-chain DvP against EURC, jurisdiction-specific selective disclosure (eWpG, MiCA), and an automated 24/7 market with daily settlement.

### Requirements

- Hide volumes, prices, positions to prevent front-running and competitive intelligence gathering
- Selective disclosure for eWpG, MiCA, and crypto-registry verification
- Atomic same-chain DvP with minutes-level finality
- Pre-trade privacy for RFQ flows and order routing
- Coupon and lifecycle events run privately

### Constraints

- Production timeline of 1-2 years with proven infrastructure
- Cross-chain DvP requires trusted relayer or bridge; out of scope for primary path
- Must compose with existing crypto-registries and custodial infrastructure
- Deployment must satisfy ICMA BDT data-model expectations for bond identifiers

## Approaches

### UTXO Shielded Notes

```yaml
maturity: production
context: i2i
crops: { cr: high, o: yes, p: full, s: high }
uses_patterns: [pattern-shielding, pattern-private-stablecoin-shielded-payments, pattern-icma-bdt-data-model, pattern-regulatory-disclosure-keys-proofs]
poc_spec: pocs/private-bond/custom-utxo/SPEC.md
example_vendors: [paladin, railgun]
```

**Summary:** Bonds modelled as UTXO notes with hidden amount and owner; transfers via JoinSplit ZK circuits; per-note viewing keys for selective disclosure.

**How it works:** A note commits to (asset, amount, owner_pk) via a Pedersen or Poseidon hash and lives in a Merkle tree. Spending a note publishes a nullifier and creates new commitments through a zero-knowledge proof; the verifier contract checks Merkle membership, nullifier uniqueness, and balance preservation. Issuance is a global note split; redemption is a burn proof. Identity is dual: a transport address (gas, KYC) plus a shielded keypair (spending + viewing).

**Trust assumptions:**
- L1 consensus and the verifier contract
- Gas relayer for liveness on private withdrawals
- Issuer for the global-note-to-holder-notes split at issuance
- No per-circuit trusted setup (UltraHonk uses a universal KZG SRS)

**Threat model:**
- A circuit or verifier soundness bug (e.g., an under-constrained circuit) lets an attacker forge or double-spend notes; metadata leakage at deposit/withdraw boundaries is the practical privacy exposure
- Loss of local note state (spending key or note secrets) is unrecoverable and collapses to lost funds
- Relayer or issuer censorship (withheld inclusion, or gated KYC entry); mitigated by direct withdraw at the cost of address linkage

**Works best when:**
- Production maturity matters and proven infrastructure (Railgun, Paladin) is available
- Strongest privacy is required (amounts, counterparties, and addresses through gas relayer)
- Institutional vendor support reduces in-house circuit work

**Avoid when:**
- ZK toolchain is unavailable to the issuer
- Coupon logic requires complex computation that does not map cleanly to circuits

**Implementation notes:** PoC implements UTXO with Noir / UltraHonk; multi-token transfers require same-token circuit constraints, so per-pool deployments are the working assumption. Compliance hooks via attestation-gated entry (zero-knowledge proof of KYC Merkle inclusion) and per-note viewing keys.

### Privacy L2

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns: [pattern-privacy-l2s, pattern-user-controlled-viewing-keys]
poc_spec: pocs/private-bond/privacy-l2/SPEC.md
example_vendors: [aztec, miden]
```

**Summary:** Bonds as native private notes inside a privacy-native rollup; protocol-level privacy without dedicated circuit work.

**How it works:** Aztec exposes private notes and contracts as native primitives. Bond issuance, transfer, and coupon logic run in private functions with client-side proving. Incoming Viewing Keys (IVKs) provide account-level read access; nullifier keys are app-siloed for damage containment.

**Trust assumptions:**
- Sequencer for ordering (currently centralized in early deployments)
- Bridge contract for L1 settlement
- Aztec proving system soundness

**Threat model:**
- Sequencer outage or censorship; rollup escape paths leak linkage during forced exit
- Bridge boundary leaks deposit and withdraw amounts
- IVK compromise reveals all account-level flows

**Works best when:**
- Bond logic is complex (coupons, lifecycle) and benefits from native privacy primitives
- Throughput needs justify L2 economics
- Decentralization roadmap of the rollup is acceptable to the issuer

**Avoid when:**
- Production deployment is needed before the rollup hits its decentralization milestones
- Sequencer trust is incompatible with the issuer's risk model

### co-SNARKs (MPC Coprocessor)

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-co-snark]
example_vendors: [taceo-merces]
```

**Summary:** A 3-party MPC committee holds secret-shared balances offchain and produces collaborative Groth16 proofs that the chain verifies.

**How it works:** Bond state lives offchain under MPC sharing; institutional senders submit shares, the committee computes the transition under MPC, and emits a co-SNARK on chain. Account-model simplicity is preserved at the application layer; addresses remain visible.

**Trust assumptions:**
- Honest-majority 3-party MPC committee (TACEO coNoir uses REP3 / 3-party Shamir, tolerating one corrupt node)
- Co-SNARK soundness
- Committee liveness

**Threat model:**
- Collusion of two of the three nodes breaks confidentiality
- Counterparty addresses leak. Confidentiality covers amounts alone
- Batch latency creates a settlement window

**Works best when:**
- Institutional custodial models are acceptable
- Amount confidentiality is sufficient and counterparty privacy is not required
- Throughput target matches batched proving (~200 TPS, TACEO-reported)

**Avoid when:**
- Honest-majority assumption among MPC nodes is incompatible with the threat model
- Sender or receiver anonymity is required

### FHE Coprocessor

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-private-shared-state-fhe]
poc_spec: pocs/private-bond/fhe/SPEC.md
example_vendors: [zama, fhenix]
```

**Summary:** Encrypted balances under FHE with a threshold-decryption network; computation happens directly on ciphertexts.

**How it works:** Balances are FHE ciphertexts; bond transfers and coupon arithmetic run homomorphically. A threshold network holds decryption shares; ACL grants gate read access on a per-balance basis. The chain stores ciphertexts and ACL state.

**Trust assumptions:**
- t-of-n threshold network for decryption keys
- FHE library implementation correctness
- ACL model adoption by all bond participants

**Threat model:**
- Threshold compromise reveals ciphertexts
- No revocation per ciphertext; revocation requires a re-encryption / re-grant on balance update
- Shared throughput (500-1000 TPS, vendor-reported) is a network-wide bottleneck

**Works best when:**
- Bond logic involves complex calculations (coupon accruals, derivatives) that map naturally to FHE
- Per-balance ACL granularity is required by the disclosure model
- Custodial threshold-network model is acceptable

**Avoid when:**
- High throughput is required and the deployment cannot tolerate shared-network bottlenecks
- Revocation per disclosure is a hard requirement

## Comparison

| Axis | UTXO Shielded Notes | Privacy L2 | co-SNARKs | FHE |
|---|---|---|---|---|
| **Maturity** | production | prototyped | prototyped | prototyped |
| **Context** | i2i | i2i | i2i | i2i |
| **CROPS** | CR:hi O:y P:full S:hi | CR:med O:part P:full S:med | CR:med O:part P:part S:med | CR:med O:part P:part S:med |
| **Trust model** | Self-custody (L1 + ZK) | Sequencer + bridge | Honest-majority 3-party MPC | t-of-n threshold network |
| **Privacy scope** | Amounts + addresses (via gas relayer) | Amounts + addresses (account level) | Amounts only; addresses public | Amounts only; addresses public |
| **Performance** | High gas, chain-dependent throughput | L2-internal fees, unknown TPS | ~95K gas/tx batched, ~200 TPS (vendor) | ~300K gas/tx, 500-1000 TPS shared (vendor) |
| **Operator req.** | No (gas relayer optional) | Yes (sequencer) | Yes (MPC committee) | Yes (threshold network) |
| **Cost class** | High (L1 verify) | Low (L2-internal) | Low (batched) | Medium |
| **Regulatory fit** | Strong (per-note view keys) | Strong (IVKs, app-siloed nullifiers) | Strong for known counterparty | Strong (per-balance ACL) |
| **Failure modes** | Relayer censor; metadata at boundaries | Sequencer outage; bridge exploit | Node collusion; batch latency | Threshold compromise; no revocation per ciphertext |

## Persona perspectives

### Business perspective

For institutional bond issuance and trading at scale, UTXO Shielded Notes is the default: production maturity (Railgun ~USD 5b lifetime shielded volume as of 2026), white-label vendor coverage (Paladin), privacy over amounts, counterparties, and addresses (addresses via gas relayer), and a regulatory story built on per-note viewing keys that maps cleanly onto eWpG and MiCA disclosure regimes. Privacy L2 fits where bond logic is complex (coupons, structured lifecycle) because it removes circuit-engineering work, but the issuer must accept the rollup's decentralization timeline. co-SNARKs and FHE fit specific institutional contexts: bilateral or club-mode markets where address visibility is acceptable, or coupon-heavy products where homomorphic arithmetic is the natural model.

### Technical perspective

Engineering effort scales by approach. UTXO Shielded Notes requires deploying a verifier and shipping a ZK toolchain, but mature vendor implementations remove the circuit-design burden. Privacy L2 trades sequencer dependency for native primitives, removing in-house circuit work but adding bridge integration. co-SNARKs requires running or contracting an MPC committee with operational discipline; throughput is bounded by batch cadence. FHE simplifies the programming model for complex bond logic but inherits shared throughput limits across all FHE applications on the network. PoC results across all three EthSystems implementations confirm that selective-disclosure semantics work in each model; the architectural choice is driven by trust topology and bond-logic complexity.

### Legal & risk perspective

This is a perspective for legal review by the deploying issuer, not legal advice. The four options expose distinct disclosure interfaces: UTXO Shielded Notes via per-note viewing keys plus nullifier publication; Privacy L2 (Aztec) via account-level Incoming Viewing Keys with app-siloed nullifier keys; co-SNARKs via MPC-mediated disclosure scoped to committee membership; FHE via per-balance ACL with no per-ciphertext revocation (revocation depends on subsequent balance updates triggering re-grants). Whether any of these interfaces satisfies eWpG / MiCA disclosure expectations or the local crypto-registry's compliance interface is a question for jurisdictional review against the specific regime; the document does not assert that any option is approved by a regulator.

## Recommendation

### Default

For institutional bond issuance and trading on a 1-2 year production timeline, default to UTXO Shielded Notes with [Paladin](../vendors/paladin.md) or [Railgun](../vendors/railgun.md) as the underlying shielded pool. This is the category with documented production volume, vendor coverage, and a disclosure interface that has been mapped onto eWpG / MiCA expectations.

### Decision factors

- If bond logic is dominated by coupon and lifecycle arithmetic that is awkward in circuits, choose Privacy L2 (Aztec, Miden) and accept the rollup-decentralization timeline.
- If counterparties are bilateral or named (e.g., dealer-to-dealer), and address visibility is acceptable, choose co-SNARKs (TACEO Merces) for MPC-based disclosure with simpler programming.
- If per-balance ACL granularity is mandated by the disclosure model and complex computation is needed, choose FHE (Zama, Fhenix) and plan for shared-throughput bottlenecks.

### Hybrid

Issuance can run through UTXO with a vendor-provided shielded pool; secondary trading can route through a Privacy L2 with a bridge to the L1 pool, amortizing high-frequency trades while keeping settlement security on L1 for primary issuance and large trades. Compliance gating (KYC attestation, crypto-registry verification) sits at the boundary as an attestation-gated deposit.

## Open questions

1. **Multi-Asset Bond Support.** Tranches, multiple currencies, or collateral types within shielded note systems require either per-tranche pools or extended circuit constraints; unresolved.
2. **Coupon Payment Mechanisms.** Patterns for automated, privacy-preserving coupon distribution to shielded bondholders are emergent; no canonical solution.
3. **Cross-Chain Settlement.** Beyond same-chain atomic DvP, acceptable trust models for cross-chain bond settlement (relayers, bridges, messaging) are unsettled.
4. **Secondary Market Structure.** Private RFQ systems with sufficient price discovery; unresolved at the standard level.
5. **Pre-Trade Privacy.** The boundary between order-flow privacy and post-trade confidentiality differs by market; no standard answer.
6. **Market Data & Analytics.** Bond pricing, yield curves, and analytics under transaction privacy require either trusted publishers or zk-statistics; unresolved.
7. **Regulatory Standards.** Standardization of selective-disclosure formats for eWpG / MiCA across jurisdictions is incomplete.
8. **Legacy Integration.** Bridges between on-chain privacy and traditional bond settlement (Euroclear, Clearstream, MarketAxess) are absent.

