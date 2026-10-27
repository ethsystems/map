---
title: "Pattern: Pre-trade Privacy (Encrypted Order Flow)"
status: ready
maturity: testnet
type: meta
layer: hybrid
last_reviewed: 2026-10-27

sub_patterns:
  - name: "Threshold-encrypted mempool"
    pattern: pattern-threshold-encrypted-mempool
    crops_summary: "Medium CR via committee, partial privacy (decrypted post-inclusion), low-medium security (committee collusion risk)."
  - name: "Private transaction broadcasting"
    pattern: pattern-private-transaction-broadcasting
    crops_summary: "Low-medium CR via private relay, partial privacy pre-inclusion, relies on builder honesty."
  - name: "Shielded settlement"
    pattern: pattern-shielding
    crops_summary: "Medium CR, full privacy on amounts/counterparties post-inclusion."

works-best-when:
  - RFQ or secondary trading must not leak intent, size, or price pre-inclusion.
  - Order flow needs protection from both the public mempool and builders that could front-run.

avoid-when:
  - Public mempool visibility of orders is acceptable.
  - Latency budgets cannot accommodate threshold-encryption or private builder round-trips.

context: both
context_differentiation:
  i2i: "Institutional block trades signal portfolio intent long before settlement. Encrypting RFQ and quote flow until inclusion prevents counterparties, intermediaries, and builders from adverse selection or front-running. Both sides have legal recourse if an intermediary leaks, so the threat is reputational and economic rather than adversarial."
  i2u: "Retail orders are extracted by builders, searchers, and private relays with operator-controlled visibility. Encrypted or threshold-encrypted submission removes the builder's ability to unilaterally front-run, but requires the user's tooling to integrate with the encrypted path. Without a fallback to an unencrypted route, a stalled decryption committee can effectively censor."

crops_profile:
  cr: low
  o: partial
  p: partial
  s: low

crops_context:
  cr: "Depends on who can submit to the encrypted path. Reaches `high` only if the encrypted mempool committee is selected permissionlessly via stake-weighted rotation and includes a fallback to the standard mempool when decryption stalls."
  o: "Some encryption and decryption implementations are open; some private builder stacks are proprietary. Reaches `yes` when all components, including the committee software and key-release logic, are open-source under permissive or copyleft licenses."
  p: "Hides order intent, size, and price from the public mempool and (with threshold encryption) from any individual committee member. Metadata (timing, wallet connection, submission path) remains visible unless additionally shielded."
  s: "Rides on committee honesty for threshold-encrypted paths or on operator honesty for private-builder paths. Reaches `high` with hardened collusion resistance and key compartmentalization across independent operators."

post_quantum:
  risk: medium
  vector: "Threshold encryption and commit-reveal schemes often rely on pairings (BLS) or curve-based encryption broken by CRQC."
  mitigation: "Lattice-based threshold encryption or hash-based commit-reveal with post-quantum VRFs for committee selection."

visibility:
  counterparty: [quote_contents]
  chain: [settlement_events]
  regulator: [full_order_lifecycle via audit path]
  public: []

standards: []

related_patterns:
  composes_with: [pattern-threshold-encrypted-mempool, pattern-private-transaction-broadcasting, pattern-shielding]
  see_also: [pattern-private-pvp-stablecoins-erc7573, pattern-dvp-erc7573]

open_source_implementations:
  - url: https://github.com/shutter-network/rolling-shutter
    description: "Threshold-encrypted mempool reference implementation (Shutter keyper, production on Gnosis Chain)"
    language: Go
  - url: https://github.com/flashbots/suave-geth
    description: "Private builder and confidential compute environment for order flow (archived 2025; historical reference)"
    language: "Go"
---

## Intent

Prevent front-running and information leakage by routing quotes and orders via encrypted or private submission paths so intent, size, and price stay hidden until inclusion. Settlement then proceeds on a shielded pool or confidential rail so the executed amounts also remain private.

## Components

- Encrypted submission path: either a threshold-encrypted mempool (committee holds key shares; ciphertext is decrypted only after ordering is committed) or a private-builder stack (confidential execution environment that sees the order but commits to ordering honesty).
- RFQ broker: off-chain service that routes quote requests to allow-listed counterparties and records the quote lifecycle for audit.
- Settlement rail: shielded pool or privacy L2 where the winning order settles with amounts hidden.
- Fallback path: unencrypted submission route invoked if the encrypted path stalls or fails.
- Audit trail: signed or committed record of every RFQ, quote, and settlement for compliance review.

The encrypted mempool details are covered in `pattern-threshold-encrypted-mempool`; the settlement layer in `pattern-shielding` or `pattern-privacy-l2s`.

## Protocol

1. [user] A buyer emits an RFQ off-chain. The request is routed to allow-listed counterparties via the RFQ broker.
2. [user] Counterparties return quotes privately to the broker, who relays them only to the requestor.
3. [user] The requestor selects a winning quote and submits the corresponding order via the encrypted path (threshold-encrypted ciphertext or private builder).
4. [operator] The committee or private builder includes the order in a block without revealing its contents pre-inclusion.
5. [contract] Settlement executes on the shielded rail: amounts, sender, and receiver stay hidden.
6. [auditor] The broker and committee produce an auditable record of the RFQ lifecycle for compliance review.

## Guarantees & threat model

Guarantees:

- No public mempool leakage of intent, size, or price pre-inclusion.
- Auditable RFQ lifecycle available to regulators via scoped access.
- Settled amounts remain private when the settlement rail is shielded.

Threat model:

- Honesty of the threshold committee or private builder. A colluding quorum can decrypt early or leak selectively.
- Liveness of the encrypted path. A stalled committee or builder forces the fallback path, where orders become visible again.
- RFQ broker trust. A malicious broker can leak quote flow to non-participants even if the on-chain path is encrypted.
- Counterparty allow-list correctness. Mis-listed counterparties can exfiltrate order intent legally but undesirably.

## Trade-offs

- Latency and availability are tied to the privacy routing. A slow committee or builder can miss inclusion windows.
- Additional infra dependency. Both the encrypted mempool and the RFQ broker are new services that must be operated and monitored.
- Fallback paths reintroduce public mempool visibility; protocol design must make fallback rare and auditable.
- Allow-listed counterparties create a two-sided trust boundary that must be governed.

## Example

- Three quotes are received for a block trade.
- The winning quote is submitted via the encrypted path.
- The losing quotes remain undisclosed to the public and to competing market makers.
- Settlement finalizes on a shielded pool. On-chain observers see only the settlement event, not the price or size.

## See also

- [Shutter Network docs](https://docs.shutter.network/docs/shutter/research/the_road_towards_an_encrypted_mempool_on_ethereum)
- [SUAVE overview](https://writings.flashbots.net/the-future-of-mev-is-suave) (historical: SUAVE is archived)
- [Flashbots documentation](https://docs.flashbots.net/)
