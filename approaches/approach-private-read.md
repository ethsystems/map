---
title: "Approach: Private Read"
status: draft
last_reviewed: 2026-11-15

use_case: private-read
related_use_cases: [private-oracles]

primary_patterns: [pattern-tee-based-privacy, pattern-private-information-retrieval, pattern-network-anonymity]
supporting_patterns: [pattern-tee-network-anonymity, pattern-private-transaction-broadcasting]
---

# Approach: Private Read

## Problem framing

### Scenario

An asset manager monitors twenty addresses and queries transaction history ahead of execution. Every JSON-RPC call it makes is visible to its RPC provider. Read patterns leak portfolio composition, counterparty relationships and imminent trading intent, which MEV searchers monetize and competitors harvest. The institution does not want to run its own infrastructure everywhere, and the same read path must stay private against both the RPC provider and network observers, with regulator access available as scoped disclosure rather than standing visibility.

### Requirements

- Hide query source, targets and timing from the RPC provider and node operators
- Prevent pre-transaction read patterns from enabling MEV extraction
- Keep latency compatible with trading and monitoring applications
- Work for institutions that do not run their own infrastructure
- Give regulators scoped access to query audit trails for compliance
- Keep read-path privacy consistent with private transaction paths

### Constraints

- Institutions rarely control the full network path to a node
- Public mempool and block explorers leak state inferences regardless of RPC privacy
- Trading-grade latency rules out heavyweight cryptography on the hot path
- MEV protection interacts with read privacy but does not replace it
- Cost scales with node operation, and most institutions will not bear it

## Approaches

### Institution-run nodes

```yaml
maturity: production
context: both
crops: { cr: high, o: yes, p: partial, s: medium }
uses_patterns: [pattern-network-anonymity, pattern-private-transaction-broadcasting]
```

**Summary:** The institution runs its own full or archive node and queries local state, so no third party ever sees its reads.

**How it works:** The institution operates open-source execution and consensus clients against the public peer-to-peer network. State reads resolve locally against synced data, so query content, targets and timing never leave the institution. Transaction history additionally needs a self-hosted indexer, since the execution APIs define no method that returns an address's transactions; a node alone does not answer that query. The remaining exposure is network-level: peer connections and IP visibility are handled with network-anonymity measures where needed. Monitoring systems, risk engines and trading stacks point at an internal endpoint instead of a vendor RPC.

**Trust assumptions:**
- Client software is open source and runs as published
- The institution's own operations team maintains node health
- Local state is only as fresh as the node's sync

**Threat model:**
- Transactions broadcast from the node reveal its IP as their origin; pair with [private transaction broadcasting](../patterns/pattern-private-transaction-broadcasting.md)
- History queries routed to a third-party indexer or explorer leak those reads even when the node itself is self-hosted
- A degraded or lagging node serves stale state without warning
- Operational failure of a single node becomes a single point of failure for the desk

**Works best when:**
- The institution already runs infrastructure and can keep nodes healthy
- Read volume is high enough to justify operation

**Avoid when:**
- The institution has no infrastructure function and low read volume
- Freshness demands exceed what a single sync can guarantee

CROPS rationale: CR high, because no external party participates in the read at all and nothing to permission remains. O yes: open-source clients and public peer-to-peer protocols. P partial: query content is fully private, but network-level traffic patterns remain observable without anonymization. S medium: no third-party trust, while operational failure and network correlation are the residual risks.

### TEE-hosted private RPC

```yaml
maturity: documented
context: both
crops: { cr: low, o: partial, p: partial, s: medium }
uses_patterns: [pattern-tee-based-privacy, pattern-tee-network-anonymity]
```

**Summary:** Queries are encrypted to an attested enclave operated by a private-RPC provider. The provider cannot read query content, but it sees who queries, when and how much, and it needs an oblivious state store (ORAM) to hide which state the enclave reads.

**How it works:** The provider runs its RPC serving layer inside a Trusted Execution Environment. Institutions authenticate, encrypt queries to the enclave's attested key, and receive responses; the provider sees only aggregate counters and billing metadata. Remote attestation lets the institution verify the exact code before sending anything. The provider still fronts the public network, so its own node connections follow normal visibility rules. Where the provider must not learn which consumer connects, the enclave path pairs with [TEE network anonymity](../patterns/pattern-tee-network-anonymity.md).

**Trust assumptions:**
- TEE hardware and attestation infrastructure behave as specified
- The enclave code is open and matches its measurement
- The provider's billing and access systems are separate from the enclave

**Threat model:**
- Side channels against the enclave recover query content
- The provider denies or throttles service selectively
- Traffic analysis at the provider's edge correlates request timing with institutions

**Works best when:**
- The institution cannot run its own nodes but needs provider-independent query privacy
- Latency must stay trading-grade, with no proving on the hot path

**Avoid when:**
- The threat model includes the hardware supply chain
- Policy requires cryptographic rather than hardware-rooted privacy

CROPS rationale: CR low, because the provider can deny or throttle service selectively, which is exclusion through institutional control under the rubric. O partial: enclave code can be open while the TEE stack is vendor-controlled. P partial: the provider authenticates and bills each caller, so it always sees who queries, when and how much; the enclave hides only content, and needs ORAM to hide which state it reads. S medium: hardware trust and side channels are the weak links, as with any TEE deployment.

### Obfuscated aggregate reads

```yaml
maturity: concept
context: both
crops: { cr: high, o: partial, p: partial, s: medium }
uses_patterns: [pattern-private-information-retrieval, pattern-network-anonymity]
```

**Summary:** Read patterns are hidden inside aggregate traffic and cryptographic retrieval: batching, timing randomization and PIR-style state access make individual reads unattributable.

**How it works:** Reads from many consumers are batched at a gateway, and timing is randomized within windows so request correlation fails for the systems behind the gateway; the gateway itself sees who sent each request and when. For specific record lookups, PIR-style retrieval hides which record is read, but the gateway still sees that a query was issued and how often, and network metadata stays outside PIR's scope. So the gateway logs can answer "who read, when" for batched traffic, and "which record" only for PIR reads, never both at once.

**Trust assumptions:**
- Batch composition is large enough for individual reads to drown
- Gateway software is open and honestly implements the batching policy
- PIR hides the record index only; network-level metadata is out of its scope

**Threat model:**
- Small batches or sparse windows allow timing correlation
- The gateway colludes with an MEV searcher and shares coarse features
- Proving-free batching leaks via request size distributions

**Works best when:**
- Many institutions share similar read rhythms and the gateway aggregates them
- A middle ground between self-hosting cost and vendor trust is needed

**Avoid when:**
- A single consumer's volume dominates the batch, re-identifying it
- Strict real-time latency rules out batching windows

CROPS rationale: CR high, because reads are hidden within aggregate traffic without operator permission, though the operator still gates admission. O partial: the design is open, but there are no deployments. P partial: batch membership is hidden while timing and volume leak by design. S medium: batch discipline and honest gateways carry the privacy.

## Comparison

| Axis | Institution-run nodes | TEE-hosted private RPC | Obfuscated aggregate reads |
|---|---|---|---|
| **Maturity** | production | documented | concept |
| **Context** | both | both | both |
| **CROPS** | CR high · O yes · P partial · S medium | CR medium · O partial · P full · S medium | CR high · O partial · P partial · S medium |
| **Trust model** | no third party on the read path | hardware attestation and enclave code | batch discipline and honest gateway |
| **Privacy scope** | query private; network traffic observable | query hidden from provider; edge traffic observable | reads hidden in aggregate; timing leaks |
| **Performance** | local reads, fastest | fast, no hot-path proving | batching windows add latency |
| **Operator req.** | Yes (the institution's own) | Yes (provider infrastructure) | Yes (gateway operation) |
| **Cost class** | medium to high | low to medium | high (server compute); not measured |
| **Regulatory fit** | institution holds its own logs | provider-side audit path | structural privacy complicates audit |
| **Failure modes** | stale state; single node failure | side channels; selective denial | timing correlation; batch re-identification |

## Persona perspectives

### Business perspective

The choice is mostly about who operates infrastructure. An institution with an existing infrastructure team gets the strongest and cheapest-per-read answer from running its own nodes, and keeps its logs inside its own perimeter, which regulators tend to prefer. An institution without that function buys privacy from a provider, and the TEE-hosted RPC is the shape that keeps the provider from learning anything about the reads, with the trade concentrated in hardware trust. The obfuscated-aggregate design suits a shared gateway model, where many institutions buy read privacy together and none of them dominates the batch. MEV exposure is a revenue question, not just a security one: pre-transaction reads leaking is a direct cost.

### Technical perspective

Institution-run nodes are operationally ordinary but unforgiving about freshness and failover; the desk should plan for redundant nodes and lag monitoring. The TEE RPC is the least engineering for the most privacy, and its hard questions are attestation verification in the client and side-channel posture of the hardware generation in use. Obfuscated aggregate reads are the most novel and the least proven: batching policy, window sizing and PIR parameter choices all become part of the threat model, and there is no standard implementation to adopt. Whatever the path, the read privacy design should be settled together with the private transaction path, since the two leak into each other.

### Legal & risk perspective

Regulators may want visibility into institutional queries, and the three sub-approaches differ sharply in who holds the logs: the institution itself, a provider enclave, or effectively no one. A design that structurally cannot attribute reads will need a documented, scoped disclosure route to remain usable under compliance regimes, and institutions should confirm their own recordkeeping obligations are met when audit trails live only in aggregates. Hardware-rooted privacy raises procurement questions about supply-chain assurance that pure cryptography does not. None of this is legal advice.

## Recommendation

### Default

For an institution that can run infrastructure, institution-run nodes are the default: strongest privacy, fastest reads, full control of logs, at the cost of operations. For institutions that cannot, the TEE-hosted private RPC is the default.

### Decision factors

- If the provider itself must never see query content, pick the TEE-hosted RPC over ordinary private RPC services.
- If infrastructure operation is out of reach and provider trust must be minimized, wait for or build the obfuscated aggregate model.
- If compliance requires institution-held logs, self-hosting is the only sub-approach that guarantees it.

### Hybrid

Run own nodes for the hot trading path, and route low-frequency monitoring through a TEE-hosted RPC as overflow, with network-anonymity measures on both paths.

## Open questions

1. What is the right disclosure granularity when regulators ask for institutional read logs under a structurally private design?
2. How should read-path and transaction-path privacy compose end to end, so neither leaks the other?
3. What batch sizes and window policies make aggregate read privacy robust against timing correlation in practice?
4. Can TEE attestation be verified by consumer tooling as routinely as TLS certificates are today?
