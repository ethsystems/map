---
title: "Pattern: Network-Level Anonymity"
status: ready
maturity: concept
type: meta
layer: offchain
last_reviewed: 2026-10-27

works-best-when:
  - Metadata leakage (IP, timing, query patterns) is a threat model concern.
  - Content privacy alone is insufficient; who matters as much as what.
  - Both read privacy (RPC queries) and write privacy (transaction submission) are needed.
avoid-when:
  - Threat model does not include network-level observers.
  - On-chain content privacy is the only requirement.

context: both
context_differentiation:
  i2i: "Institutions typically run dedicated nodes or relays, so the metadata threat surface is inter-institutional rather than user-to-institution. Network anonymity hides query patterns and settlement-transaction submission from counterparty infrastructure."
  i2u: "The institution itself is often the network observer: it operates the RPC endpoint, sequencer, or relay through which users submit transactions and query state. Transport-layer protection is a precondition for meaningful user privacy, even when content-layer privacy (encrypted balances, shielded transfers) is already in place."

crops_profile: n/a

sub_patterns:
  - name: "Onion routing"
    pattern: pattern-onion-routing
    crops_summary: "Medium CR, partial privacy, medium latency. Large external anonymity set; vulnerable to global passive adversaries."
  - name: "Mixnet anonymity"
    pattern: pattern-mixnet-anonymity
    crops_summary: "Medium CR, partial privacy, very high latency. Strongest resistance to traffic correlation via cover traffic."
  - name: "TEE-assisted network anonymity"
    pattern: pattern-tee-network-anonymity
    crops_summary: "Medium CR, partial privacy, low latency. Hardware trust assumption relaxes the anonymity trilemma."

related_patterns:
  composes_with: [pattern-private-transaction-broadcasting, pattern-threshold-encrypted-mempool]
  see_also: [pattern-user-controlled-viewing-keys, pattern-forced-withdrawal, pattern-modular-privacy-stack]

open_source_implementations:
  - url: https://pse.dev/projects/tor-js
    description: "Tor-in-WASM library for browser-side onion routing from dApps"
    language: "TypeScript, WASM"
  - url: https://gitlab.torproject.org/tpo/core/arti
    description: "Rust implementation of the Tor protocol"
    language: "Rust"
  - url: https://github.com/nymtech/nym
    description: "Mixnet with cover traffic"
    language: "Rust"
---

## Intent

Hide who is sending transactions or querying state at the network layer. Content-privacy patterns (ZK, FHE, MPC) hide what is in a transaction but not who submitted it: IP addresses, timing, and query patterns still leak sender identity. Network-level anonymity complements content privacy to close the metadata gap.

This is a meta-pattern. Multiple sub-patterns exist, each with different trade-offs along the anonymity trilemma (anonymity set size, latency, bandwidth overhead). No sub-pattern dominates; the right choice depends on the deployment's latency tolerance, threat model, and infrastructure constraints.

## Components

- Transport anonymity layer: a relay network, mix network, or hardware-assisted cluster that strips or obscures sender metadata before the message reaches the destination.
- Client routing or submission library: prepares the message for the chosen anonymity layer (layered encryption, mix encoding, or secret sharing).
- Destination: RPC endpoint, sequencer, or transaction mempool. The anonymity layer sits between the client and the destination.

Each sub-pattern instantiates these components differently. See `sub_patterns` in frontmatter.

## Anonymity trilemma

Any network anonymity system trades off between three properties:

- Anonymity set size: how many users your traffic blends with.
- Latency: delay introduced by the anonymity mechanism.
- Bandwidth overhead: cover traffic or padding required.

Pure-cryptographic approaches (onion routing, mixnets) must sacrifice at least one. Hardware-assisted approaches (TEE) relax the trilemma by offloading verification to hardware, but introduce a hardware trust assumption.

| Approach | Latency | Anonymity strength | Trust assumption |
| --- | --- | --- | --- |
| Onion routing | Moderate (100-500ms) | Strong | No single relay sees full path |
| Mixnet | High (seconds to minutes) | Strongest | Threshold mix nodes and cover traffic |
| TEE-assisted | Low | Medium | Client TEE and server majority |

## Guarantees & threat model

Guarantees:

- Hides sender IP, timing correlation, and query-to-identity mapping. Strength varies by sub-pattern.
- Complements content-privacy patterns. Together they hide both what and who.

Threat model:

- Pure-cryptographic approaches are vulnerable to a global passive adversary capable of correlating traffic at both ends of a circuit. Mixnets mitigate this via cover traffic; onion routing does not.
- Hardware-assisted approaches depend on TEE attestation integrity and are exposed to side-channel attacks on the underlying hardware.
- Does not hide message content. Pair with ZK, FHE, or MPC patterns for full-stack privacy.
- No Ethereum execution client natively supports any network anonymity layer as of 2026-04, unlike Bitcoin Core which has had built-in Tor support since 2016. Integration requires external tooling.

## Trade-offs

- Stronger anonymity generally means higher latency; institutional latency requirements constrain the choice.
- Operational complexity varies: onion routing has mature tooling; mixnet integration remains non-trivial; TEE-assisted is research-stage.
- Coverage must be end-to-end. Mixing network anonymity for writes with a plain-HTTPS RPC provider for reads reintroduces the metadata gap.

## See also

- [Modular Privacy Stack](pattern-modular-privacy-stack.md): where network anonymity fits in the four-layer architecture.
- [RFP: Private Reads](../rfps/rfp-private-reads.md): read-side privacy gap.
