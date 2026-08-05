---
title: "Domain: Post-Quantum Threats"
status: draft
description: "Migrate privacy and authentication to PQ-safe foundations before harvested ciphertexts can be decrypted."
---

## TLDR

- A Cryptographically Relevant Quantum Computer (CRQC) breaks ECDSA, BLS, ECDH, Groth16, PLONK/KZG; Ethereum consensus, execution, and application privacy layers all affected.
- Harvest Now, Decrypt Later (HNDL) means confidentiality-critical applications face urgency now, even though CRQC is years away.
- Hash-based and lattice-based primitives provide migration paths; STARKs are already PQ-safe.

## PQ Threat Landscape

Quantum computers threaten cryptography through two algorithms. Shor's algorithm provides exponential speedup against problems based on the structure of algebraic groups (discrete logarithms, integer factorization). Grover's algorithm provides quadratic speedup against unstructured search, effectively halving the security level of symmetric primitives and hash functions.

### Quantum Algorithms

| Algorithm                     | Speedup     | Target                                               | Effect                                                       |
| ----------------------------- | ----------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| **Shor**                      | Exponential | Discrete logarithms, integer factorization           | Fully breaks the underlying hardness assumption              |
| **Grover**                    | Quadratic   | Unstructured search (symmetric keys, hash preimages) | Halves effective security bits (e.g., 256-bit → 128-bit)     |
| **Brassard-Hoyer-Tapp (BHT)** | Moderate    | Collision finding in hash functions                  | Reduces collision resistance (e.g., 256-bit hash → ~170-bit) |

### Broken Hardness Assumptions (via Shor)

All assumptions that derive security from discrete-log or factorization structure are fully broken by Shor's algorithm.

| Hardness Assumption                                   | Quantum Route | Affected Schemes                                    | Ethereum Impact                                             |
| ----------------------------------------------------- | ------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| Elliptic-Curve Discrete Log Problem (ECDLP)           | Shor (direct) | ECDSA, EdDSA, ECDH, Schnorr                         | Transaction signing (secp256k1), stealth address derivation |
| Discrete Log over finite fields                       | Shor (direct) | DSA, Diffie-Hellman                                 | Legacy key exchange                                         |
| Computational / Decisional Diffie-Hellman (CDH / DDH) | via ECDLP     | Diffie-Hellman key exchange, ElGamal, Oblivious PRF | TLS, on-chain encryption, viewing keys                      |
| Pairing assumptions (Bilinear DH, q-Strong DH, SXDH)  | via ECDLP     | BLS signatures, Groth16, PLONK/KZG commitments      | Consensus (BLS), zero-knowledge proof systems, blob commitments         |
| Integer factorization / Strong RSA                    | Shor (direct) | RSA, RSA accumulators                               | Accumulators, RSA-based Verifiable Delay Functions          |

### Weakened Primitives (via Grover / BHT)

These primitives are not broken but require increased parameter sizes to maintain target security levels.

| Primitive Category                                                                          | Security Property    | Quantum Effect       | Mitigation                                                 | Post-Quantum Security       |
| ------------------------------------------------------------------------------------------- | -------------------- | -------------------- | ---------------------------------------------------------- | --------------------------- |
| Hash functions (SHA-256, Keccak)                                                            | Preimage resistance  | Grover: halves bits  | Use 256-bit hashes for 128-bit PQ security                 | Sufficient at current sizes |
| Hash functions (SHA-256, Keccak)                                                            | Collision resistance | BHT: reduces by ~1/3 | 256-bit hash retains ~170-bit collision security           | Sufficient at current sizes |
| Symmetric encryption (AES)                                                                  | Key security         | Grover: halves bits  | AES-256 for 128-bit PQ security (AES-128 drops to ~64-bit) | Use AES-256                 |
| Message Authentication Codes, Pseudorandom Functions, Key Derivation Functions (HMAC, HKDF) | Key security         | Grover: halves bits  | Double key sizes                                           | Use 256-bit keys            |

### PQ-Safe Foundations

Assumptions and schemes with no known quantum speedup. These form the basis for post-quantum migration.

| Assumption Family     | Hardness Problem                                                   | Representative Schemes             | NIST Status                  | Notes                                                                                |
| --------------------- | ------------------------------------------------------------------ | ---------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------ |
| Lattice-based         | Learning With Errors (LWE), Ring-LWE, Short Integer Solution (SIS) | ML-KEM (Kyber), ML-DSA (Dilithium) | Standardized (FIPS 203, 204) | Primary PQ replacement for key exchange and signatures                               |
| Hash-based signatures | One-time signatures + Merkle trees                                 | XMSS, SPHINCS+ (SLH-DSA)           | Standardized (FIPS 205)      | Stateful (XMSS) or stateless (SPHINCS+); conservative: depends on hash security assumptions |
| Hash + FRI (STARKs)   | Hash collision resistance                                          | STARK proof systems                | N/A                          | Already deployed in Ethereum L2s; PQ-safe by construction                            |

## Ethereum Layer Analysis

| Layer           | Today (Broken)            | Migration Path                                                                                                                                                                                   | Target End State                             |
| --------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- |
| **Consensus**   | BLS (pairings, Dlog)      | Replace fixed BLS with programmable verification; use hash-based sigs or STARK aggregation (see [Lean Ethereum](../patterns/pattern-lean-ethereum.md), [Lean Roadmap](https://leanroadmap.org/)) | Validators submit **proofs**, not signatures |
| **Execution**   | ECDSA (secp256k1)         | Account abstraction (EIP-8141-style); modular validation (custom sig / ZK checks)                                                                                                                | Account = **verification logic**             |
| **Application** | Groth16, PLONK, KZG, ECDH | Replace pairings & Dlog: STARKs, hash commitments, PQ KEMs                                                                                                                                       | **Hash/STARK-first stack**                   |

## Application-Layer Breakages

HNDL makes privacy migration more urgent than authentication: harvested ciphertexts can never be re-encrypted, while signature forgery is at least partially remediable through emergency coordination. New designs should use PQ key exchange for any confidentiality surface that persists beyond a single session.

Ethereum inherits PQ transport encryption for some surfaces (Go 1.24 ships hybrid PQ TLS by default for HTTPS/JSON-RPC). What it _cannot_ inherit is application-layer privacy: on-chain ciphertexts, ECDH-based key derivation, stealth address generation, ZK-proven encryption, and access pattern hiding are blockchain-specific problems with no industry equivalent.

### Anonymity and Unlinkability

| Surface           | Broken Primitive            | Solution Path                                                                                                                                                | Status          | Pattern                                                        |
| ----------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------- | -------------------------------------------------------------- |
| Stealth Addresses | ECDH key exchange           | ML-KEM + OMR sidecar (33x ciphertext bloat; [Module-LWE protocol](https://eprint.iacr.org/2025/112) ~66.8% faster scan)                                      | Active research | [Stealth Addresses](../patterns/pattern-stealth-addresses.md)  |
| zkTLS             | MPC/2PC on ECDH handshake   | Co-design MPC with ML-KEM algebraic structure; TLS 1.3 support prerequisite                                                                                  | Unsolved        | [zk-TLS](../patterns/pattern-zk-tls.md)                        |
| zkID (export)     | EC-based signatures         | Poseidon-internal hash-based PQ sigs + STARK verification on device                                                                                          | Tractable       | [ZK KYC/ML ID](../patterns/pattern-zk-kyc-ml-id-erc734-735.md) |
| zkID (import)     | NIST PQ sig arithmetization | 131x gap vs Groth16; [EIP-8051](https://eips.ethereum.org/EIPS/eip-8051)/[8052](https://eips.ethereum.org/EIPS/eip-8052) precompiles for direct verification | Open research   | [ZK KYC/ML ID](../patterns/pattern-zk-kyc-ml-id-erc734-735.md) |

### Confidentiality

| Surface                              | Broken Primitive               | Solution Path                                                                                      | Status    | Pattern                                                                                                                |
| ------------------------------------ | ------------------------------ | -------------------------------------------------------------------------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------- |
| Note discovery / viewing keys        | EC-based key derivation        | ML-KEM (outside ZK circuit) + OMR                                                                  | Tractable | [Shielding](../patterns/pattern-shielding.md), [Noir Private Contracts](../patterns/pattern-noir-private-contracts.md) |
| Proven-correct encryption to auditor | ElGamal (EC scalar mul)        | Lattice PKE outside circuit + Poseidon symmetric encryption inside circuit (detect-and-flag model) | Partial   | [Regulatory Disclosure](../patterns/pattern-regulatory-disclosure-keys-proofs.md)                                      |
| Protocol-enforced decryptability     | Proving lattice PKE in-circuit | Field mismatch (q=3,329 vs BN254); simpler than full ML-KEM but still expensive                    | Unsolved  | N/A                                                                                                                    |

### Dependencies

- **Client-side GPU proving**: PQ privacy requires on-device ZK proofs. NTT-based PQ primitives align well with GPU architecture ([client-side-gpu-everyday-ef-privacy](https://pse.dev/blog/client-side-gpu-everyday-ef-privacy)).
- **STARK migration**: SNARKs (Groth16, PLONK/KZG) broken; STARKs (hash + FRI) survive. See [ZK Proof Systems](../patterns/pattern-zk-proof-systems.md).

## Affected Patterns

| Pattern                                                                                       | PQ-Broken Primitive                | HNDL Risk | Mitigation Path                    |
| --------------------------------------------------------------------------------------------- | ---------------------------------- | --------- | ---------------------------------- |
| [Stealth Addresses](../patterns/pattern-stealth-addresses.md)                                 | ECDH (shared secret)               | High      | ML-KEM + OMR sidecar               |
| [PSI-DH](../patterns/pattern-private-set-intersection-dh.md)                                  | DDH / commutative encryption       | Medium    | Lattice-based PSI                  |
| [MPC Custody](../patterns/pattern-mpc-custody.md)                                             | Threshold ECDSA/EdDSA              | Low       | ML-DSA / hash-based threshold      |
| [TEE Key Manager](../patterns/pattern-tee-key-manager.md)                                     | ECDSA/BLS signing                  | Low       | PQ signing in TEE                  |
| [Noir Private Contracts](../patterns/pattern-noir-private-contracts.md)                       | Barretenberg (PLONK)               | High      | Hash-based commitments / STARKs    |
| [Private Shared State (co-SNARK)](../patterns/pattern-private-shared-state-cosnark.md)        | Groth16                            | Medium    | co-STARK alternatives              |
| [TEE+ZK Settlement](../patterns/pattern-tee-zk-settlement.md)                                 | Groth16/PLONK                      | Medium    | STARKs                             |
| [co-SNARK](../patterns/pattern-co-snark.md)                                                   | co-SNARK (Groth16-based)           | Medium    | co-STARK                           |
| [Threshold Encrypted Mempool](../patterns/pattern-threshold-encrypted-mempool.md)             | Pairing-based threshold encryption | Medium    | Lattice-based threshold encryption |
| [Shielding](../patterns/pattern-shielding.md)                                                 | ZK proofs (impl-dependent)         | High      | STARK-based shielded pools         |
| [vOPRF Nullifiers](../patterns/pattern-voprf-nullifiers.md)                                   | EC-based OPRF (DDH)                | Low       | Lattice-based OPRF                 |
| [Pretrade Privacy](../patterns/pattern-pretrade-privacy-encryption.md)                        | Threshold encryption               | Medium    | Lattice-based threshold            |
| [zk-TLS](../patterns/pattern-zk-tls.md)                                                       | MPC on ECDH key exchange           | Medium    | MPC/2PC over ML-KEM (unsolved)     |
| [ZK KYC/ML ID](../patterns/pattern-zk-kyc-ml-id-erc734-735.md)                                | Underlying proof system            | Medium    | STARK-based proving                |
| [ZK Proof Systems](../patterns/pattern-zk-proof-systems.md)                                   | Groth16, PLONK/KZG (pairing/Dlog) | Medium    | PLONK/IPA, STARKs, folding schemes |
| [Origin-Locked Confidential Ledger](../patterns/pattern-origin-locked-confidential-ledger.md) | ElGamal (EC-based)                 | High      | Lattice-based PKE                  |
| [Safe Proof Delegation](../patterns/pattern-safe-proof-delegation.md)                         | Recursive ZK (if EC-based)         | Medium    | STARK-based recursion              |
**PQ-safe patterns** (no mitigation needed):

- [Private Shared State (FHE)](../patterns/pattern-private-shared-state-fhe.md): lattice-based
- [Private MTP Auth](../patterns/pattern-private-mtp-auth.md): Merkle trees, hash-based
- [Plasma Stateless Privacy](../patterns/pattern-plasma-stateless-privacy.md): hash-based ZK

## See also

- [Lean Ethereum](../patterns/pattern-lean-ethereum.md): consensus-layer PQ migration
- [Native Account Abstraction](../patterns/pattern-native-account-abstraction.md): execution-layer PQ auth agility
- [ZK Proof Systems](../patterns/pattern-zk-proof-systems.md): STARK vs SNARK PQ comparison
- [Permissionless Spend Auth](../patterns/pattern-permissionless-spend-auth.md): application-layer auth agility
- [NIST PQC standards](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [Ethereum PQ tasklist (ethresear.ch)](https://ethresear.ch/t/tasklist-for-post-quantum-eth/21296)
- [How to hard-fork to save most users' funds in a quantum emergency](https://ethresear.ch/t/how-to-hard-fork-to-save-most-users-funds-in-a-quantum-emergency/18901)
- [Espalier: succinct lattice arguments for client-side proving (Palmette, 2026)](https://palmette.xyz/espalier.pdf): lattice-based transparent proof system, preprint stage
