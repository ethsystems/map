---
title: "Approach: Private Payments"
status: ready
last_reviewed: 2026-08-11

use_case: private-stablecoins
related_use_cases: [resilient-disbursement-rails, private-treasuries]

primary_patterns:
  - pattern-shielding
  - pattern-privacy-l2s
  - pattern-regulatory-disclosure-keys-proofs
supporting_patterns:
  - pattern-private-stablecoin-shielded-payments
  - pattern-private-iso20022
  - pattern-plasma-stateless-privacy
  - pattern-tee-based-privacy
  - pattern-co-snark
  - pattern-user-controlled-viewing-keys
  - pattern-verifiable-attestation
  - pattern-network-anonymity
  - pattern-forced-withdrawal
  - pattern-forward-secure-signatures
  - pattern-recipient-derived-receive-addresses
  - pattern-relay-mediated-proving
  - pattern-mesh-store-forward-submission
  - pattern-private-mtp-auth
  - pattern-proof-of-innocence
  - pattern-private-information-retrieval

pocs:
  folder: pocs/private-payment
  requirements: pocs/private-payment/REQUIREMENTS.md
  pocs:
    - name: "Shielded Pool"
      sub_approach: "L1 Shielded Payments"
      spec: pocs/private-payment/shielded-pool/SPEC.md
      status: benchmarked
    - name: "Plasma/Intmax2"
      sub_approach: "Stateless Plasma"
      spec: pocs/private-payment/plasma/SPEC.md
      status: benchmarked

open_source_implementations:
  - url: https://github.com/Railgun-Privacy/contract
    description: "Railgun shielded pool (L1, production volume)"
    language: Solidity
  - url: https://github.com/AztecProtocol/aztec-packages
    description: "Aztec privacy-native L2"
    language: TypeScript / Noir
  - url: https://github.com/InternetMaximalism/intmax2
    description: "Intmax2 stateless plasma"
    language: Rust
---

# Approach: Private Payments

## Problem framing

### Scenario

A multinational bank executes daily institutional stablecoin transfers (USDC, EURC) between subsidiaries, mid-cap counterparties, and a humanitarian arm disbursing to recipients in adversarial jurisdictions. The bank needs amount and counterparty privacy on chain, selective disclosure for regulators by jurisdiction, integration with SWIFT/ISO 20022, and a separate rail where recipients have no client-side proving capability and intermediaries may be compelled or breached.

### Requirements

- Hide amounts, counterparties, and transaction patterns on chain
- Selective disclosure to regulators per jurisdiction (MiCA, GENIUS, etc.)
- Integration with USDC/EURC and ISO 20022 / SWIFT messaging
- Predictable cost for high-frequency operations
- Off-ramp unlinkability for the disbursement rail
- Survives partner compromise, recipient device seizure, and successor-regime inheritance

### Constraints

- Must compose with existing custody and KYC workflows
- L1 verification gas at current shielded-pool sizes is costly per transfer
- Adversarial-jurisdiction rail has no client-side ZK, no reliable internet, EAL4+ smartcard envelope
- Organizational separation between identity-layer operator and field partner is mandatory for the disbursement rail

## Approaches

### L1 Shielded Payments

```yaml
maturity: prototyped
context: both
crops: { cr: high, o: yes, p: partial, s: high }
uses_patterns:
  [
    pattern-shielding,
    pattern-regulatory-disclosure-keys-proofs,
    pattern-forced-withdrawal,
    pattern-network-anonymity,
    pattern-private-information-retrieval,
  ]
poc_spec: pocs/private-payment/shielded-pool/SPEC.md
example_vendors: [railgun]
```

**Summary:** Shielded ERC-20 pool on Ethereum L1 using commitment/nullifier scheme; users transfer between hidden notes.

**How it works:** Deposit moves an ERC-20 balance into a Merkle tree of commitments; transfers spend a nullifier and create new commitments via a JoinSplit-style zero-knowledge proof; withdraw burns a commitment to a destination address. A gas relayer pays gas on the destination's behalf to avoid linking funded EOAs. Selective disclosure runs through per-note viewing keys.

**Trust assumptions:**

- L1 consensus and the verifier contract
- Gas relayer is willing to relay (liveness only; not custodial)
- No per-circuit trusted setup (UltraHonk uses a universal KZG SRS)

**Threat model:**

- Adversary observes L1 directly; cannot break ZK soundness
- Relayer may censor; users can fall back to direct withdraw at the cost of address linkage
- Network-level timing correlation is unmitigated at this layer
- Third-party RPC path retrieval and note discovery leak the spent-note index; mitigable via PIR or by running own indexer

**Works best when:**

- High-value transfers where L1 settlement security justifies gas
- Anonymity-set sharing across many participants is acceptable
- Counterparty privacy via per-token shielded pools is sufficient
- End-user wallets can absorb mobile-grade proving with relayer-paid gas; the user is sovereign over funds and can self-exit independent of any operator

**Avoid when:**

- High-frequency low-value transfers where gas dominates
- Amount confidentiality must be cryptographically hidden across all observers (current designs leak via deposit/withdraw boundaries)

**Implementation notes:** PoC implemented as a UTXO model with Noir / UltraHonk; dual-key (spending + viewing) architecture; attestation-gated entry via zero-knowledge proof of KYC Merkle inclusion (depth 20, ~1M participants). Multi-token transfers require same-token circuit constraints, so per-token pools are the working assumption.

#### Benchmarks

| Operation          | Value                      |
| ------------------ | -------------------------- |
| Gas: deposit       | ~155K                      |
| Gas: transfer      | ~181K + ~2.6M verification |
| Gas: withdraw      | ~47K + ~2.6M verification  |
| Proof gen (client) | 410-991ms                  |
| Trusted setup      | No                         |

### Privacy L2

```yaml
maturity: prototyped
context: both
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns:
  [
    pattern-privacy-l2s,
    pattern-user-controlled-viewing-keys,
    pattern-regulatory-disclosure-keys-proofs,
  ]
example_vendors: [aztec, fhenix]
```

**Summary:** Confidential transfers run inside a privacy-native rollup where state is hidden by default at the protocol layer.

**How it works:** Users post transactions with client-side zero-knowledge proofs to a privacy-native sequencer (Aztec) or use FHE-based confidential balances (Fhenix). Hidden state, encrypted memo, and account-level viewing keys give institutional readers controlled access. Bridging to L1 is the privacy boundary.

**Trust assumptions:**

- Sequencer for ordering (currently centralized in early deployments)
- Bridge contract for L1 settlement integrity
- Viewing-key custody at the institution

**Threat model:**

- Sequencer could censor; rollup escape hatches mitigate but expose linkage
- Bridge boundary leaks deposit/withdraw amounts unless paired with shielded pools on the other side
- Compromise of an institution's viewing key reveals all flows under it

**Works best when:**

- Institution needs amount + counterparty privacy at high frequency
- Cost amortization matters and L2 fees are acceptable
- Treasury accepts sequencer trust during the rollup's decentralization roadmap

**Avoid when:**

- Maximum settlement security required for each transfer
- Bridge custody risk is unacceptable
- Sequencer trust does not match the institution's risk model

### Stateless Plasma

```yaml
maturity: prototyped
context: both
crops: { cr: medium, o: partial, p: full, s: medium }
uses_patterns: [pattern-plasma-stateless-privacy, pattern-forced-withdrawal]
poc_spec: pocs/private-payment/plasma/SPEC.md
```

**Summary:** Operator-coordinated rollup posts only Merkle roots on L1; users custody their transaction history client-side.

**How it works:** Users build proofs of inclusion against operator-published roots. Operators batch transfers, generate aggregated SNARKs (Plonky2-style recursion), and post anchor data to L1. Withdrawals exit through an L1 anchor contract via an exit game; users prove sufficient balance to escape an offline operator.

**Trust assumptions:**

- Operator for liveness (block production, proof aggregation)
- Users to retain their own state for exits
- L1 anchor contract for forced exit correctness

**Threat model:**

- Operator offline or censoring forces escape-game exits with weaker privacy
- User data loss collapses to lost funds; backup architecture is load-bearing
- Operator equivocation needs fraud-proof or multi-operator dispute

**Works best when:**

- Volume is high and minimal L1 footprint matters
- Institution can run or contract user-side state custody
- Forced withdrawal as a recovery path is acceptable
- End users custody their own state by design; operator cannot censor without triggering the exit game, making the I2U privacy property structural rather than contingent

**Avoid when:**

- User-side state custody is operationally infeasible
- Exit-delay risk is not tolerable for the asset class

**Implementation notes:** PoC uses Plonky2 with operator-side recursive aggregation; client proofs run in 5.9-9.8s, operator proofs in 42-49s. PlasmaBlind (folding-scheme aggregation) is a tracked alternative.

#### Benchmarks

| Operation            | Value                        |
| -------------------- | ---------------------------- |
| Gas: deposit         | ~137K (user)                 |
| Gas: transfer        | Off-chain (operator-set fee) |
| Gas: withdraw        | ~343K (operator, amortized)  |
| Gas: batch           | ~255K (operator, amortized)  |
| Proof gen (client)   | 5.9-9.8s                     |
| Proof gen (operator) | 42-49s                       |

### TEE-Based Privacy

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: no, p: full, s: low }
uses_patterns: [pattern-tee-based-privacy]
example_vendors: []
```

**Summary:** Trusted execution enclave processes transfers privately; on-chain artefact is an attestation of correct enclave execution.

**How it works:** Settlement, matching, or custody runs inside an attested enclave (AWS Nitro, Azure Confidential Computing, Intel TDX). The chain sees only enclave-signed outputs and remote-attestation evidence. Enclave-held keys never leave hardware.

**Trust assumptions:**

- TEE vendor and remote-attestation chain
- Cloud co-tenant isolation
- Enclave software supply chain (reproducible build, signing)

**Threat model:**

- Side-channel and microarchitectural attacks against the enclave class
- Vendor compromise or compelled disclosure of attestation keys
- Enclave-software vulnerabilities expose all state to a single-party adversary

**Works best when:**

- Near-term deployment is needed and ZK proving is too costly per transfer
- Institutional trust model already includes hardware vendors (HSMs, secure elements)
- Counterparties accept hardware-rooted attestation

**Avoid when:**

- Threat model includes nation-state-class side-channel adversaries
- Attestation chain centralization is a hard fail

### MPC-Based Privacy

```yaml
maturity: prototyped
context: i2i
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-co-snark]
example_vendors: [taceo-merces]
```

**Summary:** MPC nodes jointly compute transfers under secret-shared balances; co-SNARKs commit a verifiable summary on chain.

**How it works:** Bilateral counterparties send secret-shared inputs to an MPC committee that runs the transfer logic and produces a collaborative SNARK. The chain verifies the SNARK; counterparty addresses are public, but amounts and balance state are hidden under sharing.

**Trust assumptions:**

- Honest majority among MPC nodes (committee size and threshold are config-time choices)
- Co-SNARK soundness
- Liveness of the MPC committee

**Threat model:**

- Collusion above the threshold reveals all state
- Counterparty addresses leak; only amount confidentiality is provided
- MPC committee liveness is an availability boundary

**Works best when:**

- Bilateral or club-mode settlement among known counterparties
- Address-level privacy is not a goal; amount confidentiality is enough
- Operating an MPC committee is feasible

**Avoid when:**

- Sender or receiver anonymity is required
- Honest-majority assumption is incompatible with the threat model

### Resilient Disbursement Rails

```yaml
maturity: prototyped
context: i2u
crops: { cr: high, o: yes, p: full, s: high }
uses_patterns:
  [
    pattern-forward-secure-signatures,
    pattern-recipient-derived-receive-addresses,
    pattern-relay-mediated-proving,
    pattern-mesh-store-forward-submission,
    pattern-shielding,
    pattern-private-mtp-auth,
    pattern-network-anonymity,
    pattern-proof-of-innocence,
    pattern-forced-withdrawal,
    pattern-verifiable-attestation,
  ]
example_vendors: []
```

**Summary:** Composition for humanitarian disbursement under adversarial-jurisdiction threat model: forward-secure smartcard signing, recipient-derived destinations, relay-mediated proving, mesh delivery, settlement over a shielded pool.

**How it works:** Funder atomically shields the round total and emits a multisig-signed header. The recipient's EAL4+ smartcard signs an offline ECDSA voucher under a per-epoch key bound to the round and an on-card derived destination. A companion device encrypts to a relay's current key and ships via Briar (Bluetooth + Tor) or Meshtastic (LoRa), fanning out to k of N relays. A relay generates the cohort-membership SNARK with submitter binding (defeats proof-stealing front-running) and submits via Tor/Nym. The claim contract verifies and unshields to the destination, where the recipient redeems through a local agent network.

**Trust assumptions:**

- IResilientIdentity operator and implementing partner are distinct legal entities, jurisdictions, and personnel
- Relay set meets size and jurisdictional-diversity floors (pilot N≥8 / ≥2 operators / ≥2 jurisdictions; production N≥16 / ≥4 operators / ≥3 jurisdictions)
- Smartcard secure-element vendor and applet supply chain
- Mesh transport availability for last-mile

**Threat model:**

- Every party may be compelled, breached, or inherited by a hostile successor
- Recipient device may be seized; forward-secure keys block retroactive identification
- Off-ramp KYC pivot bounded by association-set k-anonymity and relay diversity
- Within-epoch per-relay linkability bounded but not cryptographically eliminated

**Works best when:**

- Compelled-partner transfer, successor-regime inheritance, or off-ramp KYC pivot are documented threats
- No central beneficiary list at the funder layer is acceptable
- Donor policy admits multi-jurisdiction relay operators

**Avoid when:**

- Recipients have reliable internet and modern wallet capability (use the institutional rails: L1 Shielded, Privacy L2, or Stateless Plasma, with viewing keys instead)
- Single-jurisdiction deployment cannot meet relay-set floors

**Implementation notes:** Production dependencies: Tor, Briar, Meshtastic, EAL4+ secure elements (Keycard-class), Noir + UltraHonk for ECDSA-in-SNARK at the relay. Cohort attestation uses ECDSA-pubkey leaves so the smartcard signs ECDSA only; in-circuit Poseidon hashing happens at the relay. Rotating EOAs funded via shielded unshield handle relay submission.

## Comparison

| Axis               | L1 Shielded                             | Privacy L2                          | Stateless Plasma                                     | TEE                              | MPC                                 | Resilient Disbursement                                   |
| ------------------ | --------------------------------------- | ----------------------------------- | ---------------------------------------------------- | -------------------------------- | ----------------------------------- | -------------------------------------------------------- |
| **Maturity**       | prototyped                              | prototyped                          | prototyped                                           | documented                       | prototyped                          | documented                                               |
| **Context**        | both                                    | both                                | both                                                 | i2i                              | i2i                                 | i2u                                                      |
| **CROPS**          | CR:hi O:y P:part S:hi                   | CR:med O:part P:full S:med          | CR:med O:part P:full S:med                           | CR:med O:no P:full S:lo          | CR:med O:part P:part S:med          | CR:hi O:y P:full S:hi                                    |
| **Trust model**    | L1 + relayer liveness                   | Sequencer + bridge                  | Operator + L1 anchor                                 | TEE vendor + supply chain        | Honest-majority MPC                 | Multi-relay + smartcard + IResilientIdentity             |
| **Privacy scope**  | Anonymity (amounts may leak)            | Amounts + counterparties + patterns | Amounts + counterparties (commitments only on chain) | Full inside enclave              | Amounts only; counterparties public | Forward-secure + off-ramp unlinkable                     |
| **Performance**    | ~181K + ~2.6M gas / 410-991ms           | L2-internal fees                    | Off-chain transfer / operator-amortized              | Variable                         | MPC overhead, batched               | Relay-mediated, mesh-bounded                             |
| **Operator req.**  | No (gas relayer optional)               | Yes (sequencer)                     | Yes                                                  | Yes (cloud)                      | Yes (MPC committee)                 | Yes (relay set with floors)                              |
| **Cost class**     | High (L1 verify)                        | Low                                 | Lowest (amortized)                                   | Variable                         | Medium                              | Relay-mediated                                           |
| **Regulatory fit** | Strong with viewing keys                | Strong with viewing keys            | Conditional (operator audit)                         | Conditional (vendor attestation) | Strong for known-counterparty       | Architectural minimization at funder layer               |
| **Failure modes**  | Relayer censor; verification gas spikes | Sequencer outage; bridge exploit    | Operator offline; user data loss                     | Side-channel; vendor compromise  | MPC collusion; liveness             | Relay set under floor; smartcard supply-chain compromise |

## Persona perspectives

### Business perspective

For institutional treasury at moderate volume with standard compliance needs, the default described in `Recommendation` is the Hybrid L1/L2 composition: Privacy L2 absorbs the high-frequency intra-bank flow and L1 Shielded Payments handles high-value or anonymity-sensitive transfers. Cost is predictable, sequencer trust during rollup decentralization is the main accepted risk. Stateless Plasma is reserved for treasuries with sustained high volume that justify operator infrastructure and user-side state custody. MPC-Based Privacy fits bilateral settlement with named counterparties, where address-level privacy is not the concern. Resilient Disbursement Rails is reserved for humanitarian programs whose donor policy already names the threat model.

### Technical perspective

Engineering capacity dictates a lot. L1 Shielded Payments is the lightest integration: deploy the verifier, run a relayer, ship dual-key wallets. Privacy L2 trades sequencer dependency for amount + counterparty hiding without circuit-engineering work in-house. Stateless Plasma is the heaviest operator burden but the lowest on-chain footprint at scale. TEE is near-term and avoids client-side proving but adds a hardware trust chain that auditors must accept. MPC requires running or contracting a committee. Resilient Disbursement Rails is research-grade across applet, circuit, verifier, and claim contract; do not deploy without a deployment-gate audit and attested organizational separation.

### Legal & risk perspective

This is a perspective for legal review by the deploying institution, not legal advice. L1 Shielded Payments and Privacy L2, paired with viewing-key disclosure, expose per-jurisdiction view keys and attestation logs (EAS, ONCHAINID) as the disclosure interface; whether that interface satisfies MiCA, GENIUS Act, or another regime is a question for jurisdictional review. Stateless Plasma adds operator records as an additional disclosure surface that legal review would scope. TEE attestations are typically accepted by auditors who already accept HSM-rooted custody, but acceptance varies by regulator. MPC exposes per-counterparty audit; address-level visibility may limit use to known-counterparty contexts depending on the regulator's view. Resilient Disbursement Rails inverts the disclosure model by minimizing what each party holds; whether a humanitarian regime accepts that posture depends on the donor policy and the destination-country regulator, and legal sign-off would document the minimization and the multi-jurisdiction relay roster.

## Recommendation

### Default

For institutional treasury and payment operations at moderate volume with standard compliance, default to a Hybrid L1/L2 composition: Privacy L2 (Aztec for native confidential transfers, Fhenix for FHE-based balances) handles frequent operations; L1 Shielded Payments (Railgun-style) handles high-value transfers or anonymity-sensitive flows. Selective disclosure runs through user-controlled viewing keys plus regulator viewing keys with time-bound, threshold-controlled scope. ISO 20022 message interpreters handle SWIFT compatibility; ERC-3643 handles compliance gating where the asset is a regulated security.

### Decision factors

- If maximum settlement security is required per transfer, choose L1 Shielded Payments
- If the institution already runs operator infrastructure and wants the lowest L1 footprint at scale, choose Stateless Plasma
- If counterparties are bilateral and known, choose MPC-Based Privacy
- If near-term deployment is required and hardware trust is already accepted, choose TEE-Based Privacy
- If recipients are in adversarial jurisdictions with no client-side ZK, choose Resilient Disbursement Rails

### Hybrid

Operate L1 Shielded Payments and Privacy L2 in tiers, bridging via cross-tier mechanisms; route high-value transfers through L1 and frequent operations through L2. Pair both with Forced Withdrawal so funds remain recoverable when sequencer or operator paths fail. For programs that span institutional treasury and humanitarian disbursement, run Resilient Disbursement Rails as a separate rail under distinct legal entities and relay rosters; do not multiplex it onto the institutional rail.

## Open questions

1. **Stablecoin Issuer Integration.** Attestation-gated entry via zero-knowledge proof of KYC works as a compliance gate; freeze and denylist integration inside shielded pools is unresolved.
2. **Liquidity Fragmentation.** Per-token shielded pools are the working assumption; cross-pool atomic swaps and multi-asset circuits remain open.
3. **Operational Recovery.** Dual-key architecture handles balance inspection under cold-storage spending keys; full business continuity workflows and key rotation under shielding are unresolved.
4. **Cross-Jurisdiction Disclosure Standards.** No common selective-disclosure format across MiCA / GENIUS / national regimes.
5. **Traditional Rail Standards.** Technical standards for SWIFT / ISO 20022 integration with privacy infrastructure are emergent.
6. **Verification Gas Viability.** At ~2.6M gas per on-chain verification, the volume threshold above which L2 amortization is mandatory needs measurement per asset class.
7. **Network Timing Correlation.** Acceptable latency overhead for network-anonymity mitigations is unresolved.
8. **Relay Economic Recovery (Resilient Disbursement Rails).** Commission, L2 settlement, or funder reimbursement each carry distinct privacy consequences; unresolved.
9. **Audit-Friendly View-Key Extension (Resilient Disbursement Rails).** Recipient-derived destinations have no view-key split; can a view-only credential be added without an interactive sender step?
10. **Smartcard Supply-Chain Attestation (Resilient Disbursement Rails).** Reproducible builds, multi-party applet-key signing, perso-bureau vetting against donor-policy CC composite evaluation requirements.
11. **Cross-Cohort Metadata Leakage (Resilient Disbursement Rails).** Funder identity, cohort-size evolution, and round cadence fingerprint deployments; rotation policy unresolved.
12. **Forced-Withdrawal Interaction (Resilient Disbursement Rails).** During an L1 escape hatch, can the recipient still hit an unlinkable destination, or does forced exit collapse to a public address?
