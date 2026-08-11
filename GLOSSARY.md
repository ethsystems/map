## Glossary

### EthSystems Evaluation Frameworks

**CROPS (Censorship Resistance, Open Source and Free, Privacy, Security)**: Four non-negotiable properties for Ethereum defined by the Ethereum Foundation. The EthSystems Map evaluates solutions against all four dimensions independently. See [CONTRIBUTING.md § CROPS Evaluation](CONTRIBUTING.md#crops-evaluation) for full scoring rubrics.

**I2I (Institution-to-Institution)**: Interaction model where both counterparties are regulated entities (e.g., bank-to-bank settlement). Symmetric power dynamic; both parties have legal teams, vendor choice, and contractual recourse.

**I2U (Institution-to-End-User)**: Interaction model where one counterparty is a regulated entity and the other is an individual (e.g., retail payment). Asymmetric power dynamic; CROPS must protect the user from the institution. Institutions should be transparent to regulators and end users; end users should have the right to protect their private lives.

### Core Privacy Concepts

**Commitment**: Cryptographic value computed from hidden data (for example, amount and secrets). It lets others later verify that revealed data is consistent, without learning the data from the commitment itself.

**Note**: Private record that represents ownership of some value plus the secrets needed to prove it. The note is usually stored off-chain or encrypted; on-chain you only see commitments, nullifiers and proofs.

**Nullifier**: Unique value derived from a note’s secret and revealed when the note is spent. The system stores used nullifiers to prevent double-spending without exposing which note belonged to which party.

**Stealth Address**: is an address generated per transaction so that multiple payments to the same party cannot be easily linked on-chain. The recipient publishes some public information once; senders use it to derive fresh, unlinkable addresses.

**View Key**: is a cryptographic key that allows read-only access to encrypted state, like private balances or notes. It enables controlled visibility for auditors, regulators, or internal control functions.

**JoinSplit**: Circuit pattern that consumes one or more input notes (revealed via nullifiers) and produces one or more output notes (as new commitments). Enables private transfers, splits, and merges of value.

**Memo**: Encrypted payload attached to a private transaction containing information the recipient needs (e.g., note details, amount, blinding factor). Only the intended recipient can decrypt it using their encryption key.

### Blockchain Architecture

**Data Availability (DA)**: The guarantee that all transaction and state data needed to recompute and verify the system is actually published and retrievable. If DA fails, independent verifiers cannot reliably check state, even if proofs appear valid.

**Data Availability Layer (DA Layer)**:
A dedicated network or service that publishes and stores the data required for DA (for example, rollup or application data), separate from the main execution chain.

**Sequencer**: Layer 2 operator that accepts transactions on a L2 network, orders them, and produces blocks or batches that are later finalized on Layer 1 (like Ethereum).

**Prover**: Entity that runs a specified computation on given inputs (public and private, like L2 state transistions, private notes,...) and outputs both the result and a cryptographic proof that it was computed correctly. Provers may see plaintext data, so who runs them and how they are operated is an explicit part of the trust and privacy model.

**Verifier**: Entity (often a smart contract) that checks proofs from provers and decides whether to accept the claimed result (for example, a new state root or settlement outcome).

**Relayer**: Third party that submits transactions on behalf of users to hide identity

**Paymaster**: ERC-4337 entity that defines how gas fees for user operations are paid or sponsored. It allows us to implement controlled gasless flows or internal fee routing.

### L2 Categories

**Scaling Rollup**: ZK rollup focused on throughput/cost; state public within L2 (ZKsync, Scroll)

**Privacy Rollup**: ZK rollup designed for encrypted/private state (Aztec)

**Validium**: Validity proofs on L1; data availability off-chain

**Volition**: Hybrid model allowing per-transaction choice between on-chain and off-chain DA

### Institutional/TradFi Terms

**DvP (Delivery vs Payment)**: Atomic settlement ensuring asset delivery only if payment occurs

**PvP (Payment vs Payment)**: Atomic exchange of two payment obligations

**TCA (Transaction Cost Analysis)**: Post-trade analysis of execution quality and slippage

**AoR (Audit on Request)**: Selective disclosure mechanism generating compliance reports on-demand

**RFQ (Request for Quote)**: OTC trading workflow where market makers provide quotes privately

**Best Execution**: Obligation to obtain most favorable terms when executing client orders

**NAV (Net Asset Value)**: Total value of a fund's assets minus liabilities. Per-share NAV = total NAV / shares outstanding.

### Standards & Protocols

**[ERC-3643](https://eips.ethereum.org/EIPS/eip-3643)**: Ethereum standard for permissioned tokenized securities with built-in compliance framework

**[CMTAT](https://cmta.ch/standards/cmta-token-cmtat)**: CMTA's open standard for tokenizing financial instruments, using a rule-engine and allowlist model rather than an identity registry. Blockchain-agnostic, with reference implementations on EVM, Tezos, Solana, and a privacy-preserving Aztec/Noir variant

**[ERC-7573](https://ercs.ethereum.org/ERCS/erc-7573)**: Standard for conditional cross-chain settlement coordination

**[EIP-6123](https://eips.ethereum.org/EIPS/eip-6123)**: Ethereum standard for derivatives contracts with automated lifecycle management

**[EIP-5564](https://eips.ethereum.org/EIPS/eip-5564)**: Stealth address standard for unlinkable payments

**[EIP-7805](https://eips.ethereum.org/EIPS/eip-7805)**: Fork Choice Inclusion Lists (FOCIL) standard for censorship resistance through committee-enforced transaction inclusion

**[EIP-7701](https://eips.ethereum.org/EIPS/eip-7701)**: Native account abstraction standard enabling custom account validation logic, institutional key management, and ZK-based privacy systems

**[ERC-7945](https://ercs.ethereum.org/ERCS/erc-7945)**: Standard for confidential token transfers using cryptographic commitments to hide balances and amounts

**[ERC-8065](https://ercs.ethereum.org/ERCS/erc-8065)**: ZK token wrapper standard enabling privacy for existing ERC-20 tokens through shielded wrapping

**[ISO 20022](https://www.iso20022.org/)**: International messaging standard for financial services communication

**[ICMA BDT](https://www.icmagroup.org/market-practice-and-regulatory-policy/repo-and-collateral-markets/legal-documentation/global-master-repurchase-agreement-gmra/)**: International Capital Market Association Bond Data Taxonomy for standardized bond information

### Privacy Technologies

**FHE (Fully Homomorphic Encryption)**: Cryptographic technique allowing computation on encrypted data

**Zero-knowledge Proof**: A proof that reveals no more information than the validity of the statement it supports.

**SNARK/STARK**: Zero-knowledge proof systems (Succinct Non-interactive Arguments of Knowledge/Scalable Transparent Arguments of Knowledge)

**Co-SNARK**: Collaborative zero-knowledge proofs where multiple parties jointly prove properties

**Shielded Pool**: Privacy mechanism using cryptographic commitments to hide transaction details

**Confidential Contract**: Smart contract that operates on encrypted state while maintaining verifiability

**Circom/Groth16**: Popular zero-knowledge proof system and domain-specific language

**PLONK**: Zero-knowledge proof system with universal trusted setup

**TEE (Trusted Execution Environment)**: Hardware-based secure computation environment

**MPC (Multi-Party Computation)**: Cryptographic technique for joint computation without revealing inputs

**OPRF (Oblivious Pseudorandom Function)**: Cryptographic protocol where a server evaluates a pseudorandom function on a client's input without learning the input, and the client learns the output without learning the server's key. Used for private set intersection, password-hardening, and privacy-preserving authentication.

**PSI (Private Set Intersection)**: Cryptographic protocol that allows two parties to compute the intersection of their private sets without revealing elements outside the intersection. Variants include DH-based (commutative encryption via ECDH), OT/OPRF-based (oblivious transfer with cuckoo hashing), circuit-based (garbled circuits for arbitrary functions over matched elements), and FHE-based (homomorphic comparison).

**vOPRF (Verifiable OPRF)**: Extension of OPRF where the server provides a proof that the output was computed correctly using a committed key, preventing malicious servers from returning arbitrary values. See [RFC 9497](https://www.rfc-editor.org/rfc/rfc9497.html) for the IETF standard.

### Identity & Compliance

**PCD (Proof-Carrying Data)**: Data bundled with a cryptographic proof of its own correctness, enabling portable and composable verifiable credentials.

**Sybil Resistance**: Preventing a single actor from creating multiple fake identities to gain disproportionate influence in systems that distribute value, votes, or access.

**DKIM (DomainKeys Identified Mail)**: Email authentication standard where mail servers sign outgoing messages.

**ONCHAINID**: Decentralized identity system used by ERC-3643 for KYC/eligibility verification

**KYC/AML**: Know Your Customer/Anti-Money Laundering regulatory compliance requirements

**Attestations**: Cryptographically signed claims about identities, credentials, or data that can be verified on-chain with minimal disclosure. See [Pattern: Attestation Verifiable On-Chain](patterns/pattern-verifiable-attestation.md) for implementation approaches including EAS, W3C Verifiable Credentials, and ONCHAINID.

**EAS (Ethereum Attestation Service)**: One implementation of on-chain attestation protocol. See attestations pattern for holistic overview.

**Crypto-Registry**: Regulatory registry for digital asset compliance (eWpG requirement)

**Merkle Tree**: Cryptographic data structure for efficient membership proofs

### Regulatory Frameworks

**eWpG**: German Electronic Securities Act regulating tokenized securities

**MiCA**: EU Markets in Crypto-Assets regulation

**GENIUS Act**: US legislative framework for digital asset regulation

**SEC Rule 2a-7**: US Securities and Exchange Commission rule governing money market funds, specifying liquidity requirements, portfolio quality, maturity limits, and conditions for liquidity fees and redemption gates

**ESMA MMFR (Money Market Fund Regulation)**: EU regulation establishing rules for money market funds including daily/weekly maturity limits, stress testing obligations, and reporting to national competent authorities

### Post-Quantum Cryptography

**CRQC (Cryptographically Relevant Quantum Computer)**: Quantum computer capable of running Shor's algorithm at production key sizes, breaking ECDLP, RSA, and pairing-based assumptions.

**HNDL (Harvest Now, Decrypt Later)**: Attack where an adversary records encrypted data today to decrypt once a CRQC is available. On-chain ciphertexts are immutably stored and capturable by anyone.

**ML-KEM**: NIST PQ key encapsulation (FIPS 203, formerly Kyber). Lattice-based; replaces ECDH.

**ML-DSA**: NIST PQ signature scheme (FIPS 204, formerly Dilithium). Lattice-based; replaces ECDSA/EdDSA.

**SLH-DSA**: NIST hash-based PQ signature (FIPS 205, formerly SPHINCS+). Security relies only on hash function properties.

**Poseidon**: Arithmetic-friendly hash function for efficient ZK circuit evaluation. No known practical quantum break beyond generic hash-model considerations (e.g., Grover-style security reduction).

### Infrastructure

**Oracle**: External data provider for blockchain smart contracts

**Custodian**: Financial institution responsible for safeguarding digital assets
