---
title: "Vendor: Privacy Pools"
status: ready
maturity: production
---

# Privacy Pools – Smart Contract Protocol for Compliance-Friendly Privacy

## What it is

Privacy Pools is a smart contract protocol that extends the mixer model by allowing users to prove that their withdrawals come from (or do not come from) deposits belonging to a specific **association set**.  
Instead of merely unlinking deposits and withdrawals, users can provide **membership** or **exclusion** proofs, enabling them to disassociate from illicit deposits while retaining privacy within compliant sets.

## Fits with patterns

- [Private ISO 20022](../patterns/pattern-private-iso20022.md)
- [Regulatory Disclosure Keys & Proofs](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Proof of Innocence](../patterns/pattern-proof-of-innocence.md)
- [Shielding](../patterns/pattern-shielding.md)

## Not a substitute for

- Not a full rollup or scalability solution (operates as an L1 smart contract).
- Not an identity system (needs external KYC/credential providers for association sets).
- Not a universal compliance layer (policies depend on set construction).

## Architecture

- **Execution model**: Similar to Tornado Cash (commitments + nullifiers in a Merkle tree).
- **Association sets**: Subsets of deposits defined by a root `R_A`; users prove membership of their deposit in both the global tree and the association set.
- **Proof system**: zkSNARKs (Groth16).
- **Association Set Providers (ASPs)**: Off-chain or on-chain entities that define and publish sets (e.g. “exclude OFAC-sanctioned deposits”, “only KYC’d deposits”).
- **Data availability**: commitments and proofs recorded on-chain; full sets can be stored off-chain or on IPFS.

## Privacy domains

- **Membership proofs**: “My withdrawal came from one of these known-good deposits.”
- **Exclusion proofs**: “My withdrawal did not come from these known-bad deposits.”
- **Bilateral direct proofs**: User can still reveal the exact link to a counterparty if required.
- **Sequential proofs**: Coins can pass through multiple hands, with re-proofing at each step.

## Enterprise demand and use cases

- **Regulated exchanges / custodians**: accept deposits if proven not linked to sanctioned addresses.
- **Institutions**: segregate “clean” funds from illicit ones without sacrificing user privacy.
- **Policy pilots**: cited in academic and policy contexts (Vitalik Buterin, Fabian Schär, Ameen Soleimani et al., 2023 SSRN paper) as a way to align privacy with AML compliance.

## Technical details

- Proof circuit: double Merkle-branch check (global root + association set root).
- Association set construction can follow rules:
  - _Inclusion-based_: only low-risk deposits included.
  - _Exclusion-based_: all except high-risk deposits.
  - _Hybrid_: KYC tokens, proof-of-personhood, AI-based scoring, etc.
- Users retain the option to re-prove against updated sets or to disclose directly to a counterparty.

## Strengths

- Provides a **separating equilibrium**: honest users can prove compliance, dishonest cannot.
- Flexible: different jurisdictions can define different association sets.
- Voluntary proofs: no mandatory global allowlisting or centralized backdoors.

## Risks and open questions

- Privacy depends on **set size and accuracy**: small or poorly curated sets reduce anonymity.
- Requires trust in Association Set Providers (risk of manipulation or censorship).
- No native cross-chain support.
- Governance of set definitions is unresolved; 0xbow operates the ASP for the Ethereum mainnet deployment, live since March 2025.

## Links

- [Privacy Pools Core (0xbow) GitHub](https://github.com/0xbow-io/privacy-pools-core)
- [Original Privacy Pools research repo (ameensol)](https://github.com/ameensol/privacy-pools)
- [SSRN Paper: Blockchain Privacy and Regulatory Compliance](https://ssrn.com/abstract=4563364)
- [Vitalik blog on Privacy Pools](https://vitalik.eth.limo/general/2023/09/06/privacy.html)
