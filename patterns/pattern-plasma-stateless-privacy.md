---
title: "Pattern: Stateless Plasma Privacy"
status: ready
maturity: testnet
type: standard
layer: L2
last_reviewed: 2026-06-18

works-best-when:
  - High transaction volume with privacy requirements.
  - Users can manage their own transaction data (institutional clients with infrastructure).
  - Minimal L1 data footprint is critical for cost or privacy.

avoid-when:
  - Users cannot reliably store or back up their own state.
  - Real-time settlement finality is required (Plasma exits have delay).
  - Complex smart-contract logic is needed (stateless model limits programmability).

context: both
context_differentiation:
  i2i: "Institutions already run reliable off-chain infrastructure, so data custody is a known cost. The value of stateless Plasma at I2I scale is minimal L1 footprint and no shared pool state, which avoids revealing portfolio sizes via gas patterns or commitment volumes. Forced exits via the L1 anchor let either counterparty settle unilaterally if the block producer stalls."
  i2u: "End users cannot reliably custody their own data long-term. This pattern is only safe for user-facing deployments when paired with redundant self-hosted or trust-minimized Data Availability. Forced withdrawal is critical: the L1 anchor must accept unilateral exit proofs so a censoring block producer cannot freeze user funds."

crops_profile:
  cr: medium
  o: partial
  p: full
  s: medium

crops_context:
  cr: "Reaches `high` when the L1 anchor enforces forced exits via cryptographic proof-based withdrawals that bypass block producer liveness. Drops to `low` if the only withdrawal path depends on a single block producer's cooperation."
  o: "Reaches `yes` when the block producer software, circuit code, and client software are all published under permissive or copyleft licenses in forkable repositories."
  p: "Transaction amounts, sender, and receiver are hidden from chain observers; only block commitments and per-block sender lists are visible on L1. Network-layer metadata (IP, timing against the block producer) remains out of scope."
  s: "Rides on L1 security for deposit and exit, on the zero-knowledge proof system for transfer validity, and on the user's ability to preserve their own data. Reaches `high` with post-quantum hash-based ZK primitives and robust self-hosted Data Availability."

post_quantum:
  risk: medium
  vector: "Depends on the proof system. FRI-based systems (e.g., Plonky2) are hash-based and already PQ-leaning; KZG-based systems inherit elliptic-curve exposure. BLS signature aggregation on block producers is classically broken by CRQC."
  mitigation: "FRI or STARK-based transfer validity proofs plus hash-based or lattice-based signature aggregation on the block producer."

visibility:
  counterparty: [amounts, identities]
  chain: [block_commitments, sender_lists]
  regulator: [full_tx with user-provided viewing material]
  public: [block_commitments]

standards: [ERC-20]

related_patterns:
  requires: [pattern-shielding]
  alternative_to: [pattern-privacy-l2s]
  see_also: [pattern-forced-withdrawal, pattern-l2-encrypted-offchain-audit]

open_source_implementations:
  - url: https://github.com/ethsystems/pocs/tree/master/pocs/private-payment/plasma
    description: "EthSystems PoC scaffolding an Intmax2-style stateless Plasma private-payment stack"
    language: "Rust, Solidity"
---

## Intent

Use a stateless Plasma architecture to enable private token transfers where transaction data stays with users client-side, only commitments are posted on-chain, and validity is proven via zero-knowledge proofs. This provides strong transaction-graph privacy with L2 scalability, at the cost of moving data-availability responsibility to users.

## Components

- L1 anchor contract: stores block commitments (Merkle roots of transaction hashes) and handles deposits, withdrawals, and forced exits.
- Block producer: aggregates transactions, collects signatures, and posts the block commitment to L1. Stateless with respect to transaction contents.
- Client-side prover: users generate ZK balance and transfer proofs locally (e.g., recursive FRI-based proofs).
- User-held Data Availability: users custody their own note and transfer history. Optional trust-minimized DA Layer for redundancy.
- Forced-exit mechanism: L1 contract accepts exit proofs independently of the block producer, bypassing liveness failure.

## Protocol

1. [user] Deposit an ERC-20 to the L1 anchor contract, which credits a balance commitment on L2.
2. [user] Generate a zero-knowledge proof of a valid transfer client-side; send the proof and an encrypted note to the recipient.
3. [operator] The block producer collects signatures and proofs, builds a Merkle tree of the new state, and verifies correctness.
4. [operator] The block producer posts the state root to L1 as a minimal block commitment.
5. [user] The recipient stores the received note locally and can spend it in future transactions.
6. [user] To exit, generate an exit proof showing ownership and initiate L1 withdrawal (subject to a challenge period).
7. [contract] If the block producer stalls, any user can initiate forced exit against the L1 anchor using their own locally held proofs.

## Guarantees & threat model

Guarantees:

- Transaction amounts, sender, and receiver are hidden from chain observers; only commitments and per-block sender lists are visible.
- zero-knowledge proofs ensure no double-spend or inflation without revealing transaction details.
- Users control their own data; no operator can freeze specific balances if forced exit is implemented.
- Funds are secured by L1; users can always exit with a valid proof.

Threat model:

- Soundness of the zero-knowledge proof system.
- Correct implementation of the L1 anchor, especially the forced-exit path.
- User data loss. A user who loses their local transaction history loses access to their funds; there is no protocol-level recovery.
- Block producer liveness. A stalled producer halts new transactions; users can still exit unilaterally with their own proofs.
- Network-layer metadata (timing, IP against the block producer) is out of scope for the pattern.

## Trade-offs

- User responsibility for data backup. Institutions can absorb this cost; individual users often cannot without additional infrastructure.
- Exit delay. Plasma withdrawals have a challenge period (typically around 7 days) to allow fraud proofs.
- Limited programmability. The stateless model restricts complex contract interactions compared with privacy rollups.
- Block producer trust for liveness. Censorship is possible even if theft is not.
- Client compute. Proof generation requires client-side resources, which is acceptable for institutions but may be demanding for end-user devices.

## Example

- Institution A deposits 1m stablecoin to the L1 anchor and receives a private balance commitment.
- Institution A transfers 500k privately to Institution B, generates the proof locally, and sends an encrypted note.
- The block producer includes the transaction in a block and posts only the Merkle root to L1.
- Institution B receives the note, verifies the proof, and stores it locally.
- On-chain observers see only that a deposit occurred and that a state root was updated; no amounts and no parties.
- Institution B can later withdraw to any L1 address, breaking the link to the original depositor.

## See also

- [Plasma original paper](https://plasma.io/)
- [Intmax2 paper (eprint 2023/1082)](https://eprint.iacr.org/2023/1082)
- [Post-Quantum Threats](../domains/post-quantum.md)
