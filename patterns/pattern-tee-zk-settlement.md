---
title: "Pattern: Hybrid TEE + ZK settlement"
status: ready
maturity: research
type: standard
layer: hybrid
last_reviewed: 2026-10-21

works-best-when:
  - Counterparty risk is the primary concern and atomic settlement is the goal.
  - A privacy breach (the enclave operator or hardware vendor sees trade details) is an acceptable residual risk that can be bounded by contractual controls.
  - Settlement coordination requires lower latency than client-side zero-knowledge proving can achieve.
avoid-when:
  - Trustless guarantees are required; prefer client-side zero-knowledge proving.
  - Regulators reject opaque enclave execution even with downstream compliance layers.
  - A simple same-chain shielded transfer is sufficient without matching or coordination.

context: i2i

crops_profile:
  cr: medium
  o: partial
  p: partial
  s: medium

crops_context:
  cr: "Censorship resistance sits at `medium` because settlement ultimately anchors on L1, but the enclave operator controls order intake and message flow, so it can delay or drop submissions before on-chain anchoring."
  o: "Smart contracts and proof systems are typically open source, while the enclave binary, attestation infrastructure, and operational tooling often ship from a single vendor with limited forkability."
  p: "Privacy is `partial`: counterparties, amounts, and matching logic are hidden from network observers, but the enclave operator and hardware vendor can observe plaintext during execution. The zero-knowledge proof bounds financial risk but does not hide data inside the enclave."
  s: "Security rides on TEE attestation integrity, correct nonce-monotonicity enforcement inside the zero-knowledge circuit, and honest state-root anchoring on L1. Side-channel attacks on the underlying hardware are a known exposure."

post_quantum:
  risk: high
  vector: "A CRQC can forge settlement proofs based on pairing-friendly curves (Groth16, PLONK with KZG). Orders encrypted to an elliptic-curve enclave key and encrypted state blobs held by enclave operators face Harvest-Now-Decrypt-Later risk, and a CRQC can recover the ECDH secrets behind published stealth-address announcements, linking outputs to recipients."
  mitigation: "STARK-based settlement proofs with hash commitments, and post-quantum key exchange for order encryption. See [Post-Quantum Threats](../domains/post-quantum.md)."

standards: [EIP-5564, ERC-20, ERC-3643]

related_patterns:
  requires: [pattern-tee-based-privacy, pattern-zk-proof-systems]
  composes_with: [pattern-stealth-addresses, pattern-shielding, pattern-dvp-erc7573]
  alternative_to: [pattern-co-snark]
  see_also: [pattern-tee-key-manager, pattern-cross-chain-privacy-bridge]

open_source_implementations: []
---

## Intent

Coordinate private settlement inside a Trusted Execution Environment while anchoring correctness with a zero-knowledge proof verified on-chain. The enclave acts as a neutral synchronization layer between parties who do not trust each other: it matches orders, manages escrow state, and produces a succinct proof of correct execution. External observers see only commitments, proofs, and stealth-address outputs, while the enclave operator sees plaintext under contractual terms.

The key property is the separation of financial risk from privacy risk. The zero-knowledge proof attests that the state transition follows the rules the circuit checks, and the on-chain verifier rejects any settlement without a valid proof, so a compromised enclave cannot settle a transition the circuit rejects. The keys for both legs are revealed together (on one network, in the transaction that accepts the proof), but settlement itself is not atomic on any network: after the timeout, refunds compete with unclaimed notes (see Trade-offs). If the enclave is compromised, trade details leak, but the circuit and the timeout path keep locked funds from being lost. In institutional settings, this residual privacy exposure is managed through contractual controls (non-disclosure agreements, audit rights, penalties) with the enclave operator.

## Components

- Enclave keypair and attestation report: generated inside the Trusted Execution Environment and verified by each counterparty before submission.
- Matching engine inside the enclave: decrypts orders, validates nonces against state, and matches counterparties according to settlement rules.
- Stealth-address escrow: notes are locked to per-settlement stealth addresses with dual spending conditions (stealth-derived key or original-owner timeout path) enforced by the zero-knowledge circuit.
- Zero-knowledge prover inside the enclave: generates the succinct proof of correct matching and monotonic state transition.
- On-chain state commitment contract and verifier: stores state roots and checks each settlement proof.
- Announcement contract: publishes the ephemeral stealth-address keys atomically with settlement acceptance.
- Off-chain encrypted state blobs and key management: persist state across enclave restarts; replication covers enclave-node failure.

## Protocol

1. [operator] The enclave generates a keypair, publishes an attestation report, and exposes the public key so counterparties can verify the report and encrypt orders.
2. [user] Each counterparty encrypts its order (asset, amount, conditions, nonce) to the enclave public key and submits it on-chain or through a direct channel.
3. [operator] The enclave decrypts orders, validates nonces against state to prevent rollback, and matches counterparties according to the settlement rules.
4. [contract] Notes are locked to per-settlement stealth addresses with the zero-knowledge circuit enforcing dual spending conditions (stealth-derived key or original-owner timeout).
5. [prover] The enclave generates a zero-knowledge proof attesting that orders were valid, matching followed the protocol, state transition is monotonic, and output commitments are correct.
6. [contract] The on-chain verifier checks the proof; on success the enclave atomically publishes the ephemeral stealth-address keys to the announcement contract.
7. [user] Recipients scan the announcement contract, derive their spending keys, and claim their notes. If the enclave fails before revealing keys, original owners reclaim after the timeout.

## Guarantees & threat model

Guarantees:

- Counterparties, amounts, and matching logic are hidden from network observers; inside the enclave the operator can see plaintext under contractual terms.
- Settlement correctness: the on-chain verifier accepts only state transitions that the circuit proves valid, whether or not the enclave is intact. The proof covers only what the circuit checks: confidentiality, liveness, and the choice of which orders reach matching still depend on the enclave and its operator.
- Atomic reveal: the keys for both legs are revealed together (on one network, in the transaction that accepts the proof). Settlement itself is not atomic on any network: after the timeout, refunds compete with unclaimed notes, so each recipient has to claim in time (see Trade-offs).
- Rollback safety: nonce monotonicity is enforced inside the circuit and anchored to an on-chain state root.
- Liveness floor: the timeout refund circuit ensures original owners can reclaim locked notes if the enclave goes offline.
- Auditability: viewing keys can disclose scoped data to regulators without exposing the full state.

Threat model:

- Soundness of the zero-knowledge proof system and correctness of the on-chain verifier.
- Attestation integrity: counterparties must verify the hardware signature, the code measurement, and the revocation status before encrypting orders.
- Semi-honest enclave operator: a fully malicious operator can leak plaintext, reorder, delay, or drop messages, but cannot settle a state transition that the circuit rejects, because the proof is verified on-chain.
- Side-channel attacks on the underlying hardware (Spectre-family, cache-timing) can leak partial data even when attestation succeeds.
- Physical memory-bus attacks by whoever hosts the hardware: low-cost DRAM interposers have read enclave memory or forged attestation on Intel SGX, Intel TDX, and AMD SEV-SNP ([Battering RAM](https://batteringram.eu/) and [WireTap](https://wiretap.fail/), 2025; [TEE.fail](https://tee.fail/), 2025; [DDRop](https://research.google/pubs/ddrop-generic-memory-interposer-attacks-on-confidential-vms-by-dropping-ddr5-writes/), 2026). Intel and AMD treat these attacks as outside their threat models, so a valid attestation report does not exclude an operator with physical access.
- Metadata leakage: packet sizes, timing, and connection patterns can reveal trade magnitudes and enable front-running; padding and constant-time processing are required.
- Supply-chain tampering on the hardware is out of scope for the protocol and must be handled at sourcing time.

## Trade-offs

- Privacy depends on enclave integrity while financial correctness does not; this is the core design trade-off versus client-side zero-knowledge proving, which provides both at the cost of latency.
- Settlement atomicity is timeout-based rather than cryptographic, on one network or several. The refund path checks only the timeout, so a recipient who has not claimed by then can lose its leg to the original owner's refund while the other leg has already been claimed.
- Performance overhead versus native execution (implementation-dependent), plus enclave memory limits that constrain batch sizes on some hardware.
- Multiple enclave nodes with replicated encrypted state are required to avoid a single point of failure; otherwise one crash locks all in-flight settlements until the timeout path triggers.
- Common anti-patterns (plaintext inputs or outputs, missing nonce monotonicity, shared encryption keys, in-memory-only state) silently weaken the guarantees and must be explicitly guarded against.

## Example

A bank in Frankfurt wants to sell euro-denominated corporate bonds to a bank in Singapore for stablecoins, with both assets living on shielded pools on the same L1. Both banks verify the enclave's attestation report and encrypt their orders to the enclave public key. The enclave decrypts, validates nonces, confirms the compliance attestations, and matches the orders. Notes are locked to stealth addresses with a 24-hour timeout refund. The enclave generates a zero-knowledge proof of correct matching and submits it on-chain; once the proof verifies, the enclave publishes the ephemeral keys to the announcement contract. The buying bank derives the spending key for the bond notes and the selling bank derives the key for the stablecoin notes. If the enclave crashes between matching and proof submission, both banks reclaim their original notes through the timeout circuit.

## See also

