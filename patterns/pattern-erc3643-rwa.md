---
title: "Pattern: ERC-3643 Tokenized RWAs"
status: ready
maturity: production
type: standard
layer: L1
last_reviewed: 2026-09-29

works-best-when:
  - Regulatory compliance is mandatory.
  - Transfers must be gated by on-chain identity verification.
  - Tokenizing real-world assets such as bonds, equity, or funds.
avoid-when:
  - Full ERC-20 interoperability with unrestricted transfers is required.
  - High-frequency trading with minimal compliance checks.

context: both
context_differentiation:
  i2i: "Between institutions, centralized token-agent control (freeze, force-transfer, blacklist) is partially mitigated by bilateral legal agreements between issuer and investor. Both sides have KYC-aligned identities, legal recourse, and symmetric trust; the compliance framework enforces a pre-agreed contractual envelope rather than replacing negotiated terms."
  i2u: "End users face freeze and force-transfer risk with no negotiating leverage and no legal parity with the issuer. A user-facing deployment should restrict agent powers to compliance-triggered actions, decentralize claim issuance (for example via permissionless attestations), and pair identity checks with zero-knowledge proofs so user PII never appears on chain."

crops_profile:
  cr: none
  o: partial
  p: none
  s: medium

crops_context:
  cr: "Investor-initiated transfers are gated by the identity registry and compliance modules. The standard also gives owner- and agent-controlled administrative paths such as freezing and forced transfer, so censorship resistance is structurally `none` without strong governance constraints."
  o: "Standard specification is open and the reference implementations (T-REX, ONCHAINID) are open source under GPL-3.0, but claim issuer ecosystems are gatekept. Could reach `yes` by requiring copyleft licensing on third-party compliance modules and a permissionless attestation registry for claim issuers."
  p: "Identities and transfer parameters are public on chain. Could reach `partial` by replacing on-chain identity checks with zero-knowledge proofs of claim validity, enabling transfer validation without exposing PII."
  s: "Rides on correctness of the compliance modules and operational security of the token-agent key. Could reach `high` with multisig governance and time-locked upgrades on the issuer admin path."

post_quantum:
  risk: medium
  vector: "ECDSA signatures on agent and holder keys are broken by a CRQC. On-chain identity data is public rather than encrypted, so there is no harvest-now-decrypt-later exposure; the risk is forged agent actions and forged identity claims."
  mitigation: "Migrate agent and governance keys to post-quantum signature schemes; anchor claims via hash-based attestation schemes rather than ECDSA-signed claims."

standards: [ERC-3643, ERC-734, ERC-735]

related_patterns:
  composes_with: [pattern-crypto-registry-bridge-ewpg-eas, pattern-regulatory-disclosure-keys-proofs, pattern-zk-kyc-ml-id-erc734-735]
  see_also: [pattern-shielding, pattern-compliance-monitoring, pattern-private-mtp-auth]
---

## Intent

Enable compliant tokenization of real-world assets with built-in identity management, transfer restrictions, and regulatory rules enforced at the smart-contract level. Investor-initiated transfers are gated by on-chain identity and configurable compliance checks; issuance and agent-controlled actions follow separate rules.

## Components

- Permissioned token contract (ERC-3643) exposes an ERC-20 interface and applies identity and compliance checks to investor-initiated transfers.
- Token owner configures token metadata, registries, compliance settings, and the agents responsible for operational administration.
- Token agents perform operational controls such as minting, burning, recovery, freezing, and forced transfer, subject to the deployed implementation and governance policy.
- On-chain identity contract per participant stores claims (KYC, accreditation, jurisdiction) and exposes verification endpoints.
- Identity registry maps wallet addresses to identity contracts and gates who is eligible to hold the token.
- Compliance module suite is a pluggable rules engine that evaluates per-transfer restrictions (caps, lockups, eligibility classes).
- Claim issuers sign claims written into identity contracts and are represented on chain by claim-issuer contracts; the Trusted Issuers Registry lists which issuers the token trusts for which claim topics.

## Protocol

1. [user] Create an on-chain identity and collect signed claims from trusted issuers (KYC, accreditation, jurisdiction).
2. [operator] Deploy the permissioned token with a specific compliance ruleset and transfer restrictions.
3. [operator] Populate the identity registry with eligible participants and their identity contracts.
4. [user] Initiate an investor transfer to a recipient address.
5. [contract] For the investor-initiated transfer path, validate the relevant identities and compliance rules; revert on any failure.
6. [contract] Execute the balance change and emit transfer and compliance events. Issuance, recovery, freezing, and forced-transfer paths use their own role and eligibility checks.
7. [regulator] Query the on-chain compliance history to reconcile against regulatory filings.

## Transfer-path note

ERC-3643 distinguishes investor-initiated transfers from administrative actions. The canonical specification states that `mint` and `forcedTransfer` can bypass compliance rules while still requiring a verified recipient. Implementations can differ: the current ERC-3643 reference contract invokes `canTransfer` for `mint` but not for `forcedTransfer`. Integrators should verify the exact deployed version before treating the token as enforcing one uniform rule on every movement of value.

## Confidentiality boundary

This pattern treats transaction-level confidentiality as out of scope. For designs that add it, the boundary runs between two layers:

- **The policy layer can be reused.** Claim topics, the Trusted Issuers Registry and ONCHAINID claims define rules without requiring any particular data to be public. ONCHAINID claims already carry `signature` and `data` byte fields that can hold a zero-knowledge proof, which hides a claim's content but not its existence.
- **The execution layer cannot be made confidential without interface changes:**
  - `balanceOf` and `Transfer` events, kept for ERC-20 compatibility
  - the identity registry's `identity(address)` and `isVerified(address)`, which publish the wallet-to-identity mapping
  - the compliance contract's `canTransfer(from, to, amount)`, which takes both parties and the amount in the clear

A confidential design therefore reuses the policy layer and replaces the execution layer, for example with shielded notes and a membership proof in place of the identity registry (see [Private Client Authentication for Institutional EOAs](pattern-private-mtp-auth.md)). The same split applies to DS Protocol (see Trade-offs): its compliance configuration is policy, while the registry's public investor and country views and the per-transfer investor lookup are execution.

## Guarantees & threat model

Guarantees:

- Every investor-initiated `transfer` or `transferFrom` path passes identity verification and compliance checks before execution.
- Transfer rules can enforce KYC/AML status, investor accreditation, and jurisdictional restrictions automatically, subject to the configured claim issuers and compliance modules.
- Administrative actions such as freezes and forced transfers are observable on chain, but their authority and policy constraints must be documented separately.
- Interface compatibility with ERC-20 tooling. Transfer restrictions sit outside the ERC-20 interface, but callers can pre-check them through the identity registry's `isVerified` and the compliance contract's `canTransfer`.

Threat model:

- Trusted claim issuers: compromised issuers can mint false claims, enabling ineligible holders to pass compliance checks.
- Token-agent key compromise is catastrophic: the attacker can freeze, force-transfer, or seize any balance.
- On-chain identity links all token activity to KYC data; a privacy leak at the claim issuer side cascades to on-chain positions.
- Out of scope: transaction-level confidentiality. Amounts, positions, and counterparties remain visible on chain.

## Trade-offs

- More complex than plain ERC-20 tokens; requires identity infrastructure and claim issuer onboarding.
- Additional compliance checks on every transfer raise gas costs.
- Not suitable for permissionless DeFi composition. Many protocols will reject permissioned tokens.
- Compliance rules must be maintained and updated as regulations evolve, which requires ongoing governance.
- CMTAT covers the same intent through a rule-engine and allowlist model instead of an identity registry with claim issuers; it is blockchain-agnostic (EVM, Tezos, Solana) and has an existing privacy-preserving implementation in Noir for Aztec, relevant where transaction-level confidentiality is a goal.
- Securitize's DS Protocol covers the same intent with a different architecture. The token resolves its registry service, compliance service, compliance configuration, lock manager, wallet manager and trust service through a service registry, and investors are identified by an issuer-assigned investor ID rather than an ONCHAINID contract. Tokenized funds on Ethereum use it. Two generations are in production: the omnibus-wallet mechanism was removed upstream in July 2025, and deployments that predate the change still carry it.

## Example

An issuer tokenizes a bond as a permissioned token with investor accreditation requirements. Qualified institutional investors complete KYC and register identity contracts. Bond tokens are distributed to verified investors through the identity registry. Secondary trading is restricted by the compliance module to other qualified investors meeting the issuance rules. All transfers enforce regulatory requirements without manual oversight.

## See also

- [Private Bonds Approach](../approaches/approach-private-bonds.md)
- [ERC-3643 documentation](https://docs.erc3643.org/erc-3643)
- [CMTAT (CMTA Token) standard](https://cmta.ch/standards/cmta-token-cmtat)
- [DS Protocol (Securitize)](https://github.com/securitize-io/dstoken) and the [omnibus removal commit](https://github.com/securitize-io/dstoken/commit/e406ee17322ee1c1dc3fef669bf9a63fd479e5dc)
