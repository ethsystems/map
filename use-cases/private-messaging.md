---
title: Private Messaging
status: ready
primary_domain: Identity & Compliance
secondary_domain: Trading
---

## 1) Use Case

Encrypted communication channels for financial institutions conducting blockchain transactions. Pre-trade negotiation, deal structuring, and counterparty communication contain sensitive information that must remain confidential between parties while supporting regulatory oversight and audit requirements. The solution requires end-to-end encryption with selective disclosure capabilities.

## 2) Additional Context

## 3) Actors

Banks · Trading Desks · Compliance Officers · Regulators · Auditors

## 4) Problems

### Problem 1: Pre-Trade Communication Confidentiality

Deal negotiation messages contain sensitive pricing, terms, and counterparty information that must remain confidential between parties while supporting regulatory oversight.

**Requirements:**

- **Must hide:** Message content, negotiation history, counterparty identities (from third parties)
- **Public OK:** Existence of communication channel (metadata minimized), message delivery confirmation
- **Regulator access:** Selective disclosure for investigations; message retention for compliance; audit trail with access logging

**Constraints:**

- Regulatory record-keeping requirements (MiFID II, Dodd-Frank)
- Integration with existing communication infrastructure
- Message retention periods (typically 5-7 years)
- eDiscovery and legal hold capabilities

### Problem 2: Post-Trade Documentation Privacy

Settlement instructions, confirmation messages, and reconciliation communications contain sensitive transaction details.

**Requirements:**

- **Must hide:** Specific transaction terms, settlement details, error/exception handling
- **Public OK:** Settlement completion status
- **Regulator access:** Transaction audit trail, exception monitoring

**Constraints:**

- STP (straight-through processing) requirements
- Integration with settlement systems
- Multi-party coordination (custodians, clearing, etc.)

## 5) Recommended Approaches

See detailed solution architecture and trade-offs in [**Approach: Private Institutional Messaging**](../approaches/approach-private-messaging.md): anchored off-chain messaging as the practical default (encrypted counterpart channels, periodic integrity anchors, scoped disclosure keys), mixnet-routed messaging when counterparty graph secrecy matters, and on-chain encrypted channels for deal milestones where no intermediary may hold the record.

## 6) Open Questions

- What changes from existing systems (e.g., Bloomberg messaging) when moving to blockchain?
- Is on-chain messaging necessary, or is off-chain with on-chain anchoring sufficient?
- How do retention and eDiscovery requirements interact with encryption?

## 7) Notes And Links

- Related: Pre-trade negotiation in [private-bonds.md](private-bonds.md), [private-fx.md](private-fx.md), [private-derivatives.md](private-derivatives.md)
- Related: [private-identity.md](private-identity.md) (identity for message authentication)
