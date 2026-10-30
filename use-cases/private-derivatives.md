---
title: Private Smart Derivatives
status: ready
primary_domain: Funds & Assets
secondary_domain:
---

## 1) Use Case

Banks need to run ERC-6123 interest-rate swaps on public chains without leaking margin balances, settlement amounts, and trade terms. The solution requires all economic values and trade details to be confidential while retaining automation and auditability for institutional scale adoption.

## 2) Additional Context

## 3) Actors

Party A · Party B (swap counterparties) · Smart Derivatives Contract (SDC; ERC-6123) · Payment Token (ERC-20, to become confidential) · Valuation Oracle · Registrar/Compliance Operator · Regulator/Auditor · Keeper/Scheduler

## 4) Problems

### Problem 1: Confidential Derivatives Trading with Daily Settlement

Visible daily cashflows and buffer levels expose risk appetites and strategies, creating opportunities for adversarial pricing and front-running. Institutions need confidential margin, deltas, and trade parameters while maintaining automation and regulatory auditability.

**Requirements:**

- **Must hide:** internal SDC balances (margin), margin requirements, settlement amounts, payment amounts, daily valuation updates, optional ICMA XML trade data
- **Public OK:** counterparty identities, contract existence
- **Regulator access:** keyed per-deal view or threshold decryption for full audit trail
- **Settlement:** daily cadence; atomic, single-tx margin transfers; unwind on default
- **Ops:** chain finality in minutes acceptable; sustainable L2 cost; integrate with existing tech stacks

**Constraints:**

- Must fit existing regulatory/KYC frameworks
- Preserve ERC-6123 semantics (capped-deal behavior)
- Production timeline: **1–2 years**
- Prevent side-channel leaks (gas, calldata, mempool)

## 5) Recommended Approaches

See detailed solution architecture and trade-offs in [**Approach: Private Derivatives**](../approaches/approach-private-derivatives.md).

For the mempool side channel (pending margin and settlement transactions), see [Approach: Private Transaction Broadcasting](../approaches/approach-private-broadcasting.md).

## 6) Open Questions

- Trust model for **valuation oracle** under privacy
- How to prove **min-margin** privately
- Default handling: what leaks on **margin exhaustion** or liveness failure
- Which selective disclosure primitive fits best (view keys vs threshold decryption vs DV proofs)
- Storage for **ICMA XML** (off-chain + hash vs encrypted blob on-chain)

## 7) Notes And Links

- **Standards:** [ERC-6123](https://eips.ethereum.org/EIPS/eip-6123), [ERC-7573](https://eips.ethereum.org/EIPS/eip-7573) (optional), confidential token patterns
- **Related use cases:** [Private Bonds](./private-bonds.md): key difference is **daily cadence** and **oracle dependency**
- **Diff vs Private Bonds:** bonds = static, slow cadence; derivatives = dynamic, daily, oracle-driven
