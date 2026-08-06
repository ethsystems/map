---
title: "Survey: Privacy Standards for Institutional Ethereum"
status: draft
description: "Catalog of Ethereum standards for privacy-preserving institutional use cases, with gap analysis and decision guidance."
---

## Overview

The Ethereum ecosystem has developed multiple standards addressing institutional privacy needs—from permissioned tokenized securities to unlinkable payments and confidential transfers. This survey catalogs the current standards landscape, maps standards to use cases, identifies gaps, and provides decision guidance.

**Scope**: Standards directly relevant to institutional privacy. Excludes general-purpose token standards (ERC-20, ERC-721) unless they have privacy-specific extensions.

### TLDR for different personas

- **Business**: Standards define interoperability boundaries—choosing the right standard determines which counterparties, custodians, and service providers you can work with. ERC-3643 dominates compliant securities; ERC-7573 is emerging for settlement.
- **Technical**: Seven core standards cover most institutional privacy needs. Implementation maturity varies—ERC-3643 is production-ready, while ERC-7945/8065 are draft. Gaps exist in compliance oracle interfaces and cross-L2 coordination.
- **Legal**: Standards encode compliance assumptions—ERC-3643 embeds KYC/transfer restrictions, EIP-6123 includes regulatory reporting hooks. Verify that standard's compliance model matches your jurisdictional requirements.

## Standards Catalog

### Core Privacy Standards

| Standard | Purpose | Status | Institutional Fit |
|----------|---------|--------|-------------------|
| **ERC-3643** | Permissioned tokenized securities | Final | High |
| **ERC-7573** | Cross-chain DvP coordination | Draft | High |
| **EIP-5564** | Stealth addresses | Final | Medium |
| **EIP-6123** | Derivatives lifecycle | Final | High |
| **EIP-7805** | Fork Choice Inclusion Lists (FOCIL) | Draft (Glamsterdam) | Medium |
| **ERC-7945** | Confidential token transfers | Draft | High |
| **ERC-8065** | ZK token wrapper | Draft | High |

---

### ERC-3643: Permissioned Tokenized Securities

| Aspect | Details |
|--------|---------|
| **Purpose** | Standard facilitating compliance by enforcing transfer rules on ERC-20 functions (KYC, transfer restrictions, eligibility) |
| **Status** | Final |
| **Key Features** | On-chain identity (ONCHAINID), modular compliance rules, transfer agent controls, forced transfers for recovery |
| **Institutional Fit** | High - designed specifically for regulated securities |
| **Vendor Support** | Tokeny, multiple custodians |
| **Pattern Support** | [ERC-3643 RWA Tokenization](../patterns/pattern-erc3643-rwa.md) |
| **Limitations** | Privacy limited to access control (not cryptographic); all transactions visible to permissioned parties; limited DeFi interoperability—requires dedicated protocol deployments (e.g., [Aave Horizon](https://aave.com/docs/aave-v3/horizon)) |

**When to use**: Tokenized securities requiring regulatory compliance, transfer restrictions, and corporate actions support.

---

### ERC-7573: Cross-Chain DvP Coordination

| Aspect | Details |
|--------|---------|
| **Purpose** | Coordinate atomic settlement across networks (L1↔L2, L2↔L2) |
| **Status** | Draft |
| **Key Features** | Trade object standard, settlement coordinator interface, oracle-based condition verification |
| **Institutional Fit** | High - addresses multi-network settlement needs |
| **Vendor Support** | Emerging implementations |
| **Pattern Support** | [Atomic DvP via ERC-7573](../patterns/pattern-dvp-erc7573.md) |
| **Limitations** | Requires trusted oracle; cross-network atomicity depends on oracle reliability |

**When to use**: Settlement workflows spanning multiple chains or L2 networks; DvP with counterparties on different networks.

---

### EIP-5564: Stealth Addresses

| Aspect | Details |
|--------|---------|
| **Purpose** | Enable unlinkable payments by generating fresh addresses per transaction |
| **Status** | Final |
| **Key Features** | Meta-address publishing, ephemeral key generation, recipient scanning |
| **Institutional Fit** | Medium - useful for payments privacy, but scanning overhead at scale |
| **Vendor Support** | [Railgun](../vendors/railgun.md) implements similar concepts |
| **Pattern Support** | [Stealth Addresses](../patterns/pattern-stealth-addresses.md) |
| **Limitations** | Recipients must scan chain for payments; no built-in compliance hooks; limited to address unlinkability |

**When to use**: Payment flows where sender-recipient linkage must be hidden from chain observers; requires separate compliance layer.

---

### EIP-6123: Smart Derivative Contracts

| Aspect | Details |
|--------|---------|
| **Purpose** | Standardize derivatives lifecycle management on-chain (confirmation, valuation, margin, settlement) |
| **Status** | Final |
| **Key Features** | Trade state machine, margin management, event callbacks, termination handling |
| **Institutional Fit** | High - models institutional derivatives workflows |
| **Vendor Support** | Limited current implementations |
| **Pattern Support** | Referenced in [Atomic DvP Settlement](./approach-dvp-atomic-settlement.md) |
| **Limitations** | Privacy requires separate integration; lifecycle events visible unless combined with privacy layer |

**When to use**: OTC derivatives requiring standardized lifecycle management with clear state transitions and margin handling.

---

### EIP-7805: Fork Choice Inclusion Lists (FOCIL)

| Aspect | Details |
|--------|---------|
| **Purpose** | Censorship resistance through inclusion lists that proposers must respect |
| **Status** | Draft |
| **Key Features** | Committee-based inclusion lists, forced transaction inclusion, censorship detection |
| **Institutional Fit** | Medium-High - provides banking risk assessment confidence by guaranteeing contract state integrity; addresses censorship concerns for compliant transactions |
| **Vendor Support** | Protocol-level; no vendor implementation required |
| **Pattern Support** | [FOCIL Pattern](../patterns/pattern-focil-eip7805.md) |
| **Limitations** | L1-focused; L2 censorship resistance requires separate mechanisms |

**When to use**: Applications requiring strong censorship resistance guarantees; critical settlement transactions that must not be censored.

---

### ERC-7945: Confidential Token Transfers

| Aspect | Details |
|--------|---------|
| **Purpose** | Enable private balance and transfer amounts using cryptographic commitments |
| **Status** | Draft |
| **Key Features** | Shielded balances, confidential transfers, ZK proofs for validity |
| **Institutional Fit** | High - addresses balance/amount privacy needs |
| **Vendor Support** | Early implementations emerging |
| **Pattern Support** | [Shielding](../patterns/pattern-shielding.md) |
| **Limitations** | Draft status; implementation complexity; proving costs |

**When to use**: Token transfers where balance and amount visibility must be hidden from chain observers while maintaining verifiable correctness.

---

### ERC-8065: ZK Token Wrapper

| Aspect | Details |
|--------|---------|
| **Purpose** | Add privacy layer to existing ERC-20 tokens through ZK wrapping |
| **Status** | Draft |
| **Key Features** | Wrap any ERC-20 into shielded version, unwrap back to original, compatibility with existing tokens |
| **Institutional Fit** | High - enables privacy for existing token infrastructure |
| **Vendor Support** | Early implementations |
| **Pattern Support** | [Shielding](../patterns/pattern-shielding.md) |
| **Limitations** | Draft status; shield/unshield boundaries create linkability risks |

**When to use**: Adding privacy to existing ERC-20 tokens without replacing the token contract; gradual privacy adoption for established tokens.

---

## Standards by Use Case

| Use Case | Primary Standards | Secondary Standards |
|----------|-------------------|---------------------|
| **Private Payments** | EIP-5564, ERC-7945, ERC-8065 | EIP-7805 (censorship resistance) |
| **Tokenized Securities** | ERC-3643 | ERC-7573 (settlement) |
| **DvP Settlement** | ERC-7573, EIP-6123 | ERC-3643 (if securities) |
| **OTC Derivatives** | EIP-6123 | ERC-7573 (multi-network) |
| **Cross-Chain Operations** | ERC-7573 | EIP-7805 (L1 finality) |
| **Shielded Balances** | ERC-7945, ERC-8065 | EIP-5564 (address privacy) |
| **RWA Tokenization** | ERC-3643 | ERC-7945 (amount privacy) |

### Combination Guidance

**Securities with settlement privacy**: ERC-3643 (compliance) + ERC-7945 (amount hiding) + ERC-7573 (cross-network DvP)

**Private payments with compliance**: EIP-5564 (unlinkability) + compliance oracle (not yet standardized) + ERC-7945 (amount hiding)

**Derivatives with cross-network settlement**: EIP-6123 (lifecycle) + ERC-7573 (coordination) + oracle infrastructure

---

## Gap Analysis

### Identified Gaps

| Gap | Description | Impact | Potential Solutions |
|-----|-------------|--------|---------------------|
| **Compliance Oracle Interface** | No standard for ZK-compatible compliance attestations | Cannot combine privacy with compliance verification | ONCHAINID extension; new ERC proposal needed |
| **Selective Disclosure Coordination** | No standard for regulator view keys or audit interfaces | Manual, vendor-specific audit implementations | View key standard; audit trail ERC |
| **Cross-L2 Privacy** | No standard for private transfers between L2 networks | Privacy breaks at bridge boundaries | IBC-style private messaging; rollup coordination |
| **Institutional Key Management** | No standard for compliance-compatible custody interfaces | Each vendor implements differently | HSM/MPC interface standard |
| **Privacy-Preserving Compliance** | No standard linking ERC-3643 compliance with ZK proofs | Cannot prove compliance without revealing identity | ZK credential standards; ERC-3643 ZK extension |

### Standards at Risk

| Standard | Risk | Mitigation |
|----------|------|------------|
| **ERC-7945** | Draft; interface may evolve | Track EIP discussions; design for adapter patterns |
| **ERC-8065** | Draft; wrapping semantics may change | Monitor discussions; plan for migration paths |

---

## Decision Tree

```
Need privacy-preserving institutional transactions?
│
├─ Need compliant securities with transfer restrictions?
│  └─ YES → ERC-3643
│
├─ Need atomic settlement across networks?
│  └─ YES → ERC-7573 (+ oracle infrastructure)
│
├─ Need payment unlinkability (hide sender-recipient)?
│  └─ YES → EIP-5564 (stealth addresses)
│
├─ Need hidden balances/amounts (cryptographic privacy)?
│  ├─ New token deployment? → ERC-7945
│  └─ Existing ERC-20? → ERC-8065 (wrapper)
│
├─ Need derivatives lifecycle management?
│  └─ YES → EIP-6123
│
└─ Need censorship resistance at L1?
   └─ YES → EIP-7805 (FOCIL)
```

### Multi-Standard Architectures

| Architecture | Standards | Complexity |
|--------------|-----------|------------|
| **Simple compliant security** | ERC-3643 alone | Low |
| **Private payments** | EIP-5564 + ERC-7945 | Medium |
| **Cross-network securities DvP** | ERC-3643 + ERC-7573 | Medium |
| **Full privacy + compliance** | ERC-3643 + ERC-7945 + custom compliance oracle | High |
| **Derivatives with privacy** | EIP-6123 + ERC-7945 + ERC-7573 | High |

---

## Additional Standards

Standards not in primary scope but relevant for context:

### Related Token Standards

| Standard | Relevance to Privacy |
|----------|---------------------|
| **ERC-1400** | Security token predecessor to ERC-3643; still used in some deployments |
| **ERC-4626** | Tokenized vault standard; relevant for private vault implementations |
| **ERC-1155** | Multi-token standard; basis for some privacy-preserving NFT approaches |

### Infrastructure Standards

| Standard | Relevance to Privacy |
|----------|---------------------|
| **EIP-4337** | Account abstraction enables privacy-preserving paymasters and bundlers |
| **EIP-712** | Typed data signing used in permit-based privacy flows and meta-transactions |
| **EIP-2771** | Meta-transactions enable relayer-based privacy (hide original sender) |

### Emerging Standards

| Standard | Status | Potential Impact |
|----------|--------|------------------|
| **ERC-6551** | Final | Token-bound accounts could enable per-asset privacy policies |
| **EIP-7701** | Draft | Native account abstraction enables institutional key management with custom account rules, ZK-based privacy systems, and new private transfer constructions on L1 |
| **EIP-7702** | Draft | EOA code delegation may simplify privacy wallet implementations |

---

## Future Enhancements

This survey should be updated as the standards landscape evolves:

- [ ] Add compliance oracle interface standard when proposed
- [ ] Track cross-L2 messaging standards (IBC-like proposals)
- [ ] Monitor ERC-7945/8065 as they move toward finalization
- [ ] Document institutional key management standards when proposed
- [ ] Add ZK credential standards (e.g., for privacy-preserving KYC)
- [ ] Evaluate account abstraction privacy patterns (EIP-4337 extensions)

---

## Links and Notes

### Related Approaches

- [Atomic DvP Settlement](./approach-dvp-atomic-settlement.md) - Uses ERC-7573, EIP-6123
- [Private Bonds](./approach-private-bonds.md) - Uses ERC-3643, shielded patterns
- [Private Payments](./approach-private-payments.md) - Uses stealth addresses, shielded transfers
- [White-Label Deployment](./approach-white-label-deployment.md) - Infrastructure considerations

### Related Patterns

- [ERC-3643 RWA Tokenization](../patterns/pattern-erc3643-rwa.md)
- [Atomic DvP via ERC-7573](../patterns/pattern-dvp-erc7573.md)
- [Stealth Addresses](../patterns/pattern-stealth-addresses.md)
- [Shielding](../patterns/pattern-shielding.md)
- [FOCIL - Censorship Resistance](../patterns/pattern-focil-eip7805.md)

### External Resources

- [EIPs.ethereum.org](https://eips.ethereum.org/) - Official EIP repository
- [ERCs.ethereum.org](https://ercs.ethereum.org/) - ERC-specific repository
- [Ethereum Magicians](https://ethereum-magicians.org/) - Standards discussion forum
- [EIP-5564 Discussion](https://ethereum-magicians.org/t/eip-5564-stealth-addresses/10614)
- [ERC-3643 Specification](https://eips.ethereum.org/EIPS/eip-3643)
- [ERC-7573 Specification](https://ercs.ethereum.org/ERCS/erc-7573)
- [EIP-6123 Specification](https://eips.ethereum.org/EIPS/eip-6123)
