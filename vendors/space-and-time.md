---
title: "Vendor: Space and Time"
status: ready
maturity: mainnet
---

# Space and Time – Proof of SQL (ZK-verifiable SQL data oracle for onchain use)

## What it is

A SQL data oracle which is ZK-verifiable onchain. Space and Time allows smart contracts and API clients to run SQL queries over verifiably correct (attested), onchain events or signed, tamperproof offchain data and receive both: (a) the query result and (b) a zero-knowledge proof that the computation was performed correctly over untampered tables. 
The proof can be verified by an EVM-compatible verifier contract, enabling downstream smart contract logic to safely act on the result without trusting offchain execution or reprocessing data onchain.

## Fits with patterns

- [Pattern: Attestation verifiable on-chain](../patterns/pattern-verifiable-attestation.md)
- [Pattern: Permissioned Ledger Interoperability](../patterns/pattern-permissioned-ledger-interoperability.md)
- [Pattern: ERC-3643 Tokenized RWAs](../patterns/pattern-erc3643-rwa.md)
- [Pattern: Atomic DvP via ERC-7573 (cross-network)](../patterns/pattern-dvp-erc7573.md)

## Not a substitute for

- Price-feed oracle networks (e.g., curated feeds + aggregation + publisher diversity)
- A general-purpose Data Availability layer (it proves query correctness; DA is a separate concern)
- Full identity/compliance policy engine (you can query compliance tables, but policy definition/enforcement is up to the app)
- Not a programmable privacy rollup or App-chain (functions only as contracts on existing L1/L2)


## Architecture

- **Decentralized validator set holding tamperproof tables:** Indexed blockchain data (events, txns, balances, etc with block hashes as attestation source) as well as signed user/enterprise-loaded datasets are held by a global validator network that collectively updates commitments on each table (with BFT consensus and threshold signatures) as batches of new data are added.
- **Prover nodes:** When a contract or API issues a query, a prover executes the SQL and generates a zero-knowledge proof, then relays both to a relayer contract for verification and data callback to the client contract. This pipeline enables proofs for aggregate queries over tables with 1M+ rows in under one second.
  1. **AST:** SQL is parsed into an AST and relational plan
  2. **Witnesses:** The plan is executed to produce witness data
  3. **Commitments:** Witnesses are committed using standard polynomial commitment schemes
  4. **Constraints & Sum-check:** Constraints are generated and verified via sum-check
  5. **Openings:** Opening proofs are generated for verification
  - _More information is available in the white paper:_ https://sxtdocspub.blob.core.windows.net/docs/sxt-whitepaper.pdf
- **EVM integration (relayer + verifier):** An onchain “query relayer” endpoint + an onchain verifier contract on supported EVM networks; results are delivered to the requesting contract after verification. 
- **Developer tooling:** SDKs/repos for running queries and verifying proofs locally, and smart-contract-facing integration docs (including deployed contract addresses). 



## Privacy domains

- **Verifiability-first ZK:** The core guarantee is integrity. Proof that data wasn’t manipulated and SQL was executed correctly in a single proof. 
- **Selective disclosure by query design:** Private/regulated workflows can ask predicate-style questions (e.g., “is balance > 1000?”) and prove the boolean result without revealing the underlying sensitive value. Privacy depends on what’s stored and what the result leaks. 
- **Data governance is workload-dependent:** You can keep PII offchain and only store/verifiably query derived attributes, but the system is not inherently a privacy rollup.


## Enterprise demand and use cases

- **Target segments:** financial institutions and protocols that require smart contracts to reason over large, structured datasets with cryptographic integrity guarantees. This includes banks and asset issuers building RWA systems, stablecoin issuers, data-driven DeFi protocols, and undercollateralized lending platforms.
- **Primary users:** smart contract engineering teams and applied cryptography teams, as well as digital asset and risk groups within institutions responsible for onchain execution, policy enforcement, and data integrity.
- **Data-driven smart contracts:** contracts that depend on complex aggregations, joins, or historical analysis that are impractical to recompute onchain. Typical use cases include dynamic lending parameters, data-conditioned swap or execution hooks, and verifiable reward distribution based on historical user activity or liquidity provision.
- **Compliance and risk gating:** onchain enforcement of compliance or risk policies by querying cryptographically signed datasets (e.g., allowlists, deny lists, or risk scores) and verifying results via zero-knowledge proofs. Contracts can condition settlement or execution on proven properties of the data without exposing the underlying datasets.
- **RWAs and programmable rules:** tokenized assets whose behavior depends on external or offchain attributes (such as eligibility, valuation inputs, or lifecycle rules), while preserving integrity guarantees through signed data ingestion and verifiable query execution.

## Technical details

- **End-to-end flow:** Client contract submits query via function call to relayer contract → prover executes + generates proof → result + proof returned onchain → EVM verifier checks proof → verified result passed to client contract. 
- **Onchain verification cost target:** Verification costs as little as ~150k gas. 
- **Proof scope:** Proof attests both (1) underlying table integrity and (2) correctness of the SQL computation. For queries against EVM activity (events, balances, txns, etc.) source chain block hash is used for attestation of correctness of original data.
- **Integration surfaces:** (a) smart contract relayer/verifier contracts, (b) offchain API access for enterprises, and (c) local verification via SDK.

## Strengths

- **Performance:** The protocol is designed to execute and prove analytical SQL queries over large datasets, including aggregate queries over 1M+ rows, in under one second (see [benchmarks](https://github.com/spaceandtimefdn/sxt-proof-of-sql#benchmarks)). It can aggregate over millions of rows of indexed data within Ethereum block time on a single NVIDIA T4 GPU.
- **Cryptographic integrity at scale:** Proof of SQL scales to return zero-knowledge proofs for queries against tables with hundreds of millions of rows of indexed events/transactions without consensus-style re-execution.
- **SQL-native developer experience:** Leverage a mature query model (joins, filters, aggregates) instead of bespoke onchain indexing logic. 
- **Composable with EVM apps: **Verified results can directly gate contract execution via onchain verifier + callback pattern.
- **Works for both crypto-native and enterprise data:** Supports “join onchain + offchain” workflows (including pulling from attested sources like institutional portfolios).

## Risks and open questions

- **Data freshness / ingestion guarantees:** “Tamperproof” depends on how data is inserted, permissioned, and fingerprinted. All data queried from onchain EVM sources is attested, but offchain data can be tampered by the source.
- **Privacy is not automatic:** Predicate-style privacy is possible, but outputs can still leak if the query is misrepresented. Sensitive workloads require careful query/result design and access controls. 
- **Contract/security surface:** Verifier + relayer contracts become critical infrastructure: audits, upgrade policies, and chain-by-chain deployments matter. 


## Links

- [https://www.spaceandtime.io/](https://www.spaceandtime.io/)
- [https://github.com/spaceandtimefdn](https://github.com/spaceandtimefdn)
- [https://docs.spaceandtime.io/](https://docs.spaceandtime.io/)
- [https://staking.spaceandtime.io/](https://staking.spaceandtime.io/)
