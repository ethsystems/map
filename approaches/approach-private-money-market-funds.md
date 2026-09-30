---
title: "Approach: Private Money Market Funds"
status: ready
last_reviewed: 2026-09-30

use_case: private-money-market-funds
related_use_cases: [private-stablecoins, private-treasuries, private-rwa-tokenization]

primary_patterns:
  - pattern-shielding
  - pattern-regulatory-disclosure-keys-proofs
supporting_patterns:
  - pattern-compliance-monitoring
  - pattern-verifiable-attestation
  - pattern-erc3643-rwa
  - pattern-private-mtp-auth
  - pattern-private-shared-state-fhe
  - pattern-tee-based-privacy
  - pattern-tee-key-manager

open_source_implementations:
  - url: https://github.com/ethsystems/pocs/tree/master/pocs/private-payment/shielded-pool-compliance
    description: "EthSystems PoC: KYC-gated shielded pool with in-circuit compliance"
    language: "Noir, Solidity, Rust"
  - url: https://github.com/ethsystems/pocs/tree/master/pocs/private-bond/fhe
    description: "EthSystems PoC (draft): FHE confidential bond on Zama's fhEVM"
    language: "Solidity, TypeScript"
  - url: https://github.com/ethsystems/pocs/tree/master/pocs/private-trade-settlement/tee_swap
    description: "EthSystems PoC (draft): TEE-coordinated atomic swaps of private notes"
    language: "Rust, Noir, Solidity"
  - url: https://github.com/Railgun-Privacy/contract
    description: "Railgun shielded pool"
    language: Solidity
  - url: https://github.com/OpenZeppelin/openzeppelin-confidential-contracts
    description: "OpenZeppelin confidential contracts: ERC-7984 tokens on Zama's FHE"
    language: Solidity
---

# Approach: Private Money Market Funds

## Problem framing

### Scenario

A treasurer subscribes USD 50M USDC to a tokenized T-bill money market fund. Position size, redemption timing, and yield attribution must be hidden from competitors and other fund participants. Fund NAV and total shares outstanding stay public at the fund's publication cadence; redemptions must continue without interruption under stress. Tokenized MMFs on Ethereum today mostly keep a stable USD 1 NAV and pay yield as new shares; floating-NAV funds pay it through the share price instead.

### Requirements

- Daily or intraday NAV computation with verifiable correctness (total shares outstanding stays public)
- Aggregate figures are published at the fund's cadence, not per transaction, so totals don't reveal individual flows
- SEC Rule 2a-7 (US) and ESMA MMFR (EU) compliance: gates (MMFR only; removed from Rule 2a-7 in 2023), liquidity fees, concentration limits. Private funds follow offering rules instead, such as eligibility and investor-count limits.
- Atomic subscription and redemption settlement (no partial fills: shares and cash move together or not at all). Where the cash leg settles off-chain, the transfer agent reconciles.
- Yield attribution provably correct per investor without revealing positions

### Constraints

- Custody of disclosure keys must be administratively feasible for the transfer agent and regulators
- Periodic full-audit checkpoints must run off the critical path of subscription/redemption
- Fund-circuit hash registered immutably at deployment; circuit upgrades imply migration
- Compliance gates (Rule 2a-7 liquidity ratio, concentration limits) must be enforceable without revealing individual positions. These portfolio rules apply to the fund's assets, so investor privacy doesn't affect them; investor-side rules such as liquidity fees depend only on aggregate flows.

## Approaches

### ZK Shielded Commitments

```yaml
maturity: documented
context: i2i
crops: { cr: high, o: yes, p: full, s: high }
uses_patterns: [pattern-shielding, pattern-regulatory-disclosure-keys-proofs, pattern-verifiable-attestation, pattern-compliance-monitoring, pattern-erc3643-rwa, pattern-private-mtp-auth]
example_vendors: [paladin, railgun, privacypools]
```

**Summary:** Share positions are shielded UTXO commitments; a running `total_shares` commitment is updated per transaction; NAV opens via threshold key holders independent of the operator.

**How it works:** Each position is a commitment to (attestation hash, share count, entry NAV). Subscription mints a position commitment and increments a running Pedersen commitment to `total_shares`; redemption nullifies the position and decrements the running total. ZK circuits enforce conservation, gate logic (Rule 2a-7 liquidity ratio, concentration), and yield-attribution constraints. NAV is computed by any t-of-n threshold key holders opening `total_shares` and multiplying by an oracle price-per-share; a periodic full-audit checkpoint verifies the running total against all active positions, off the redemption critical path.

**Trust assumptions:**
- L1 / L2 consensus and verifier contract correctness
- Threshold custodian / auditor set (t-of-n) for NAV opening; operator does not participate in the threshold
- Oracle integrity for per-share price (single oracle or quorum)

**Threat model:**
- Adversary observes L1 / L2; cannot break ZK soundness
- Threshold compromise (t collusions) reveals NAV but not individual positions
- Oracle compromise distorts NAV; mitigated by quorum
- Periodic full-audit catches running-total drift from circuit bugs

**Works best when:**
- Operator independence is a hard requirement (regulator, donor policy, internal governance)
- Daily or intraday NAV cadence matches the proving budget
- Threshold custodian / auditor administration is feasible

**Avoid when:**
- Yield logic is complex enough that circuit complexity exceeds practical bounds
- Threshold administration overhead (key rotation, custodian onboarding) is unacceptable

**Implementation notes:** PoC uses Railgun-class shielded pool primitives, as in EthSystems' [shielded-pool-compliance](https://github.com/ethsystems/pocs/tree/master/pocs/private-payment/shielded-pool-compliance) PoC: a KYC-gated pool with attestation expiry, a compliance policy enforced inside the value-conserving circuits, and an encrypted audit channel to a threshold committee. An MMF variant adds issuance into investor positions, yield distribution and investor counters. Compliance gates encoded as ZK public outputs (e.g., eligibility and jurisdiction caps; portfolio rules such as post-redemption weekly liquid assets ≥ 50%, the SEC Rule 2a-7 minimum since the 2023 amendments, are measured on the fund's assets, outside the pool); regulator scope via per-position view keys logged through EAS. Yield attribution uses pro-rata share-of-total computation: each redeemer proves `my_shares / total_shares * total_yield = entitled_amount`. This fits a floating-NAV fund, and only with entry NAV tracked per position; in a stable-NAV fund, yield reaches positions as new shares, through a periodic mint into each position or a public multiplier over share-denominated positions.

### FHE Encrypted Balances

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: partial, p: partial, s: medium }
uses_patterns: [pattern-private-shared-state-fhe, pattern-compliance-monitoring, pattern-erc3643-rwa]
example_vendors: [zama, fhenix, orion-finance]
```

**Summary:** Balances are FHE ciphertexts on an FHE-enabled L2; NAV is computed homomorphically; threshold key holders decrypt for publication.

**How it works:** Subscriptions encrypt the share count under the FHE network's keys; balances are stored as ciphertexts with ACL-based read access. NAV is computed under encryption (sum of ciphertexts × per-share price); a t-of-n threshold network decrypts the result for posting on chain. Yield attribution runs as homomorphic arithmetic; gate logic uses encrypted comparisons.

**Trust assumptions:**
- t-of-n threshold network for FHE decryption keys
- FHE library implementation correctness
- ACL grants honored by all participants

**Threat model:**
- Threshold compromise reveals all balances
- No revocation per ciphertext; ACL revocation requires re-encryption on balance update
- Shared throughput across all FHE applications on the network is a bottleneck

**Works best when:**
- Yield logic is complex (path-dependent strategies) and benefits from homomorphic arithmetic
- Per-balance ACL granularity matches the disclosure model
- Threshold-network trust is acceptable to all custodians

**Avoid when:**
- Operator independence requires that no single network be load-bearing
- Throughput pressure across all FHE applications is unacceptable

### TEE Enclave

```yaml
maturity: documented
context: i2i
crops: { cr: medium, o: no, p: full, s: low }
uses_patterns: [pattern-tee-based-privacy, pattern-tee-key-manager, pattern-compliance-monitoring]
example_vendors: [inco, iexec]
```

**Summary:** Positions sealed inside a TEE enclave; NAV computed in the clear internally; remote-attested results posted on chain.

**How it works:** Subscriptions are encrypted to the enclave's attested public key; the enclave maintains positions in cleartext memory. NAV computation, gate enforcement, and yield attribution all run inside the enclave; the enclave signs results under its attested key. Selective disclosure runs through enclave-mediated views: regulators submit signed queries, the enclave returns scoped responses.

**Trust assumptions:**
- TEE vendor and remote-attestation chain (Intel TDX, AMD SEV, AWS Nitro, Azure CC)
- Cloud co-tenant isolation
- Enclave software supply chain (reproducible build, multi-party signing)

**Threat model:**
- Side-channel and microarchitectural attacks against the enclave class
- Vendor compromise or compelled disclosure of attestation keys
- Enclave software vulnerabilities expose all fund state to a single-party adversary

**Works best when:**
- Near-term deployment is needed and ZK / FHE proving costs are prohibitive
- Hardware trust is already accepted by custodians
- Enclave reproducibility and signing pipeline are operationalized

**Avoid when:**
- Threat model includes nation-state-class side-channel adversaries
- Operator independence requires that no single hardware vendor be load-bearing

## Comparison

| Axis | ZK Shielded Commitments | FHE Encrypted Balances | TEE Enclave |
|---|---|---|---|
| **Maturity** | documented | documented | documented |
| **Context** | i2i | i2i | i2i |
| **CROPS** | CR:hi O:y P:full S:hi | CR:med O:part P:part S:med | CR:med O:no P:full S:lo |
| **Trust model** | Math + threshold (t-of-n) for NAV opening | Threshold (t-of-n) decryption | Hardware vendor + supply chain |
| **Privacy scope** | Amounts + addresses (bounded by anonymity set) | Amounts only; addresses public | Amounts + addresses (inside enclave) |
| **Performance** | Constant per tx; periodic full-audit scales with positions | Heaviest compute; shared throughput | Cheapest; near-instant |
| **Operator req.** | None beyond the transfer agent (relayer optional) | Yes, beyond the transfer agent (threshold network) | Yes, beyond the transfer agent (enclave host) |
| **Interop with integrators and contract holders** | Low: replaces the token's execution layer, and contracts can't hold note secrets | Medium: address-based, and contracts can hold and move encrypted balances | Medium: through the enclave's attested interface |
| **Cost class** | Medium-high | Medium | Low |
| **Yield distribution cost** | Per position per period (mint), or one update (multiplier) | Per balance per period (homomorphic add), or multiplier | Low: computed inside the enclave |
| **Regulatory fit** | Strong (per-position view keys, EAS-logged) | Strong (per-balance ACL, no revocation) | Conditional (vendor attestation) |
| **Failure modes** | Threshold compromise; circuit bugs; oracle compromise | Threshold compromise; no revocation; throughput | Side-channel; vendor compromise; enclave outage |

## Persona perspectives

### Business perspective

For a yield-bearing tokenized treasury product where confidential investor positions and a complete transfer-agent register are the load-bearing properties, ZK Shielded Commitments is the default: positions and redemptions are private, and aggregates are published at the fund's cadence rather than per transaction. FHE suits funds whose existing integrators and contract holders must keep operating on balances, since its account model keeps addresses and hides amounts only; the trade-off is reliance on the FHE network for both decryption and throughput. TEE is a viable PoC starting point and a near-term production option for funds whose custodians already accept hardware-rooted trust.

### Technical perspective

The dominant engineering questions are yield distribution into hidden positions and the size of the anonymity set. ZK with a running `total_shares` commitment keeps per-transaction proving constant and pushes full-audit cost off the critical path; circuit complexity for gate enforcement and yield attribution is the ceiling. FHE simplifies the programming model but inherits shared-throughput limits and per-ciphertext revocation gaps. TEE eliminates both proving cost and revocation issues but introduces a hardware trust chain and side-channel surface that auditors must accept. Threshold custody administration (key rotation, custodian onboarding, threshold parameter selection) is non-trivial across all three.

### Legal & risk perspective

This is a perspective for legal review by the deploying fund operator, not legal advice. The three options expose distinct evidence patterns: ZK Shielded Commitments via per-position view keys plus ZK public outputs proving investor-side gate compliance (eligibility, jurisdiction caps; portfolio rules are measured on the fund's assets) without revealing positions, with EAS-logged disclosures as the trail; FHE via per-balance ACL granularity with no per-ciphertext revocation (revocation depends on subsequent balance updates triggering re-grants, or the disclosure model has to rely on append-only audit logs); TEE via enclave-mediated disclosure that depends on enclave-code reproducibility, multi-party signing, and the vendor governance model. Whether any of these patterns satisfies SEC Rule 2a-7 or ESMA MMFR auditor expectations is a question for jurisdictional review. Custody of disclosure keys (the transfer agent's register key, regulator viewing keys; independence and jurisdictional diversity where threshold-held) is a structural-risk parameter that legal review would weigh in any of the three.

## Recommendation

### Default

For institutional-grade private money market funds, default to ZK Shielded Commitments, with privacy by default and opt-out for holders that must stay public (reserve holders, autonomous contracts). Publish aggregates at the fund's cadence rather than per transaction, and give the transfer agent a standing register feed. Unlinkability is bounded by the confidential set: in a single-fund pool with concentrated holders it is limited, and a pool shared across funds or with the cash leg widens it. Periodic full-audit checkpoints run off the redemption critical path. Selective disclosure runs through per-position view keys logged via EAS; investor-side gate compliance is encoded as ZK public outputs. Choose FHE Encrypted Balances when existing integrators and contract holders must keep operating on balances and amount-only confidentiality is acceptable.

### Decision factors

- If existing integrators and contract holders must keep operating on balances, or per-balance ACL granularity is required, and amount-only confidentiality is acceptable, choose FHE Encrypted Balances.
- If near-term deployment is required and custodians already accept hardware-rooted trust, choose TEE Enclave as a PoC or transitional path.
- If the confidential set stays small and concentrated (single fund, few large holders), none of the three approaches delivers meaningful unlinkability; widen the set (shared pool, cash leg) or re-scope to amount confidentiality.

### Hybrid

Hold confidential positions in ZK Shielded Commitments and keep opted-out holders on the public token, with conservation spanning both. Use TEE to compute per-period yield distribution across hidden positions when it exceeds practical proving budgets, with the TEE output committed back into the ZK state at the next checkpoint. FHE Encrypted Balances serve integrator and contract-holder positions that must stay operable where amount-only confidentiality is acceptable. Compliance gating runs uniformly through the regulator-disclosure-keys pattern across all rails.

## Open questions

1. **Yield-cohort attribution (floating-NAV funds).** Balancing fungible-share privacy against the complexity of attributing yield across different entry-NAV cohorts; pro-rata is the working assumption but other models may better serve specific products. Stable-NAV funds avoid cohorts by paying yield as new shares.
2. **Peer-to-peer share trading.** Shielded MMF shares traded peer-to-peer with privately enforced NAV-based pricing; integration with shielded DvP is unresolved.
3. **Mixed-currency underlying.** Hiding currency exposure when the underlying basket spans multiple currencies; oracle and ACL design are unresolved.
4. **MMF-as-cash-equivalent.** Whether shielded MMF shares can serve as the cash leg in DvP for other instruments.
5. **Anonymity-set source.** Single fund, pool shared across funds, or shared with the cash leg, and how per-fund rules are enforced in a shared pool.
6. **Account vs note.** Amount-only confidentiality with interop, vs unlinkability with a replaced execution layer.
7. **Participation.** Privacy by default with opt-out; what each public ↔ confidential crossing leaks; a minimum confidential-set size.
8. **Transfer-agent visibility.** The minimum the register feed must carry (ownership vs flows), and constraining the ciphertext in-circuit.
9. **Yield into hidden positions.** Periodic mint vs multiplier, and whether a multiplier counts as reinvestment for reporting and tax.
10. **Custody.** Multisigs, smart accounts and contract holders; client-side vs delegated proving.
11. **Migration from deployed tokens.** In-place upgrade vs reissuance, and interop cost per integrator. Several deployed tokenized MMFs run Securitize's DS Protocol, with two generations in production (omnibus removed in the newer one); in-place upgrades must fit each generation's storage layout. Locking the deployed token and mirroring positions on a confidential ledger ([origin-locked pattern](../patterns/pattern-origin-locked-confidential-ledger.md)) makes the lock contract a nominee holder: investor counters go wrong, or the reporting that fixes them is public.
12. **Liveness under stress.** The holder's path to cash when an operator or cash venue is unavailable.
13. **Collateral (future extension).** Locking a hidden position in favour of a lender, the lender's claim on default without the borrower's secret, a confidential cash leg, and keeping the register of record accurate while shares sit in a lending pool.