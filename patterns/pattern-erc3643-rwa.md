---
title: "Pattern: ERC-3643 Tokenized RWAs"
status: ready
maturity: production
type: standard
layer: L1
last_reviewed: 2026-08-11

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
  cr: "Transfers are gated by the identity registry and compliance modules. Any token agent can freeze, force-transfer, or blacklist addresses at will, so censorship resistance is structurally `none`."
  o: "Standard specification is open and the reference implementations are source-available, but claim issuer ecosystems are gatekept. Could reach `yes` by requiring copyleft licensing on compliance modules and a permissionless attestation registry for claim issuers."
  p: "Identities and transfer parameters are public on chain. Could reach `partial` by replacing on-chain identity checks with zero-knowledge proofs of claim validity, enabling transfer validation without exposing PII."
  s: "Rides on correctness of the compliance modules and operational security of the token-agent key. Could reach `high` with multisig governance and time-locked upgrades on the issuer admin path."

post_quantum:
  risk: medium
  vector: "ECDSA signatures on agent and holder keys are broken by a CRQC. HNDL risk is moderate since on-chain identity data is public but linkable to off-chain PII."
  mitigation: "Migrate agent and governance keys to post-quantum signature schemes; anchor claims via hash-based attestation schemes rather than ECDSA-signed claims."

standards: [ERC-3643, ERC-734, ERC-735]

related_patterns:
  composes_with: [pattern-crypto-registry-bridge-ewpg-eas, pattern-regulatory-disclosure-keys-proofs, pattern-zk-kyc-ml-id-erc734-735]
  see_also: [pattern-shielding, pattern-compliance-monitoring]
---

## Intent

Enable compliant tokenization of real-world assets with built-in identity management, transfer restrictions, and regulatory rules enforced at the smart-contract level. Each transfer is gated by an on-chain identity check and a configurable compliance module before the underlying transfer executes.

## Components

- Permissioned token contract (ERC-3643) exposes an ERC-20 interface but routes every transfer through compliance and identity checks.
- On-chain identity contract per participant stores claims (KYC, accreditation, jurisdiction) and exposes verification endpoints.
- Identity registry maps wallet addresses to identity contracts and gates who is eligible to hold the token.
- Compliance module suite is a pluggable rules engine that evaluates per-transfer restrictions (caps, lockups, eligibility classes).
- Claim issuers are off-chain actors that sign claims written into identity contracts; the registry tracks trusted issuers.
- Token agent holds administrative powers: freeze, force-transfer, blacklist, supply management, compliance-rule updates.

## Protocol

1. [user] Create an on-chain identity and collect signed claims from trusted issuers (KYC, accreditation, jurisdiction).
2. [operator] Deploy the permissioned token with a specific compliance ruleset and transfer restrictions.
3. [operator] Populate the identity registry with eligible participants and their identity contracts.
4. [user] Initiate a transfer to a recipient address.
5. [contract] Validate both sender and receiver against the identity registry and run every compliance module; revert on any failure.
6. [contract] Execute the balance change and emit transfer and compliance events.
7. [regulator] Query the on-chain compliance history to reconcile against regulatory filings.

## Guarantees & threat model

Guarantees:

- Every transfer passes identity verification and compliance checks before execution.
- Transfer rules enforce KYC/AML status, investor accreditation, and jurisdictional restrictions automatically.
- Full on-chain audit trail of ownership changes, freezes, and force-transfers.
- Interface compatibility with ERC-20 tooling, with additional transfer restrictions opaque to the caller.

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

## Example

An issuer tokenizes a bond as a permissioned token with investor accreditation requirements. Qualified institutional investors complete KYC and register identity contracts. Bond tokens are distributed to verified investors through the identity registry. Secondary trading is restricted by the compliance module to other qualified investors meeting the issuance rules. All transfers enforce regulatory requirements without manual oversight.

## See also

- [Private Bonds Approach](../approaches/approach-private-bonds.md)
- [ERC-3643 documentation](https://docs.erc3643.org/erc-3643)
- [CMTAT (CMTA Token) standard](https://cmta.ch/standards/cmta-token-cmtat)
