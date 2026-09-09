---
title: "Vendor: Inco"
status: ready
maturity: production
---

# Inco - TEE co-processor enabling confidential smart contract library for EVM chains

## What it is

Inco Lightning is a Solidity library and Trusted Execution Environment coprocessor that adds encrypted data types (`euint256`, `ebool`, `eaddress`) and confidential operations to standard smart contracts on existing EVM chains. Developers import a single library and write confidential logic in Solidity; encrypted values are stored as on-chain handles while confidential computation runs inside an attested enclave network at near-native chain latency. Lightning is post-quantum secure on the privacy side, and is live on Base mainnet.

## Fits with patterns

- [TEE-Based Privacy](../patterns/pattern-tee-based-privacy.md) - confidential compute inside attested enclaves with hardware trust anchors
- [Private Shared State (TEE)](../patterns/pattern-private-shared-state-tee.md) - encrypted contract state shared across mutually distrusting parties
- [TEE Key Manager](../patterns/pattern-tee-key-manager.md) - decryption keys never leave attested enclaves
- [Shielding](../patterns/pattern-shielding.md) - confidential ERC-20 balances, transfers, and allowances
- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md) - confidential token transfers for stablecoins
- [Confidential ERC-3643](../patterns/pattern-erc3643-rwa.md) - confidential ERC-3643 for RWAs
- [User-controlled viewing keys](../patterns/pattern-user-controlled-viewing-keys.md) - programmable per-handle access control decides who may decrypt each value
- [Private Intent-Based Vaults](../patterns/pattern-private-vaults.md) - private strategy execution for DeFi

## Not a substitute for

- Anonymity: wallet addresses and transaction graph remain public. Lightning hides specific onchain states, not counterparties
- Trust-minimised confidentiality from zero-knowledge proofs. Hardware trust and cloud provider anchors remain in the model.

## Architecture

The technical architecture is a based private rollup with a consensus light client (Helios) inside the Trusted Execution Environment, enabling end-to-end verifiable and trustless compute:
- **Execution model**: contracts hold handles to encrypted values in canonical on-chain state. Confidential operations (`add`, `mul`, comparisons, `select`, encrypted randomness) are executed by a Trusted Execution Environment coprocessor network triggered by on-chain events.
- **Key management**: encryption keys never leave attested enclaves. A quorum-based network of decryption nodes handles decryption and re-encryption requests, signs results, and relays them on-chain through verified callbacks.
- **Access control**: control access on each encrypted handle are enforced at the contract level, enabling role-based access and selective disclosure to approved parties (users, auditors, contracts).
- **Verifiability**: every state transition is verifiable compute, so operators cannot forge results. The design goal is that operators can at worst halt, not steal.
- **Client side**: a JavaScript SDK encrypts inputs, manages signatures, and re-encrypts outputs for local decryption by authorised users.
- **Cryptography**: the protocol utilizes NIST-approved, post-quantum-secure schemes.

## Privacy domains

- Confidential token balances, transfer amounts, and allowances
- Encrypted application state: private identity, sealed bids, hidden game state, private votes, confidential order parameters
- Onchain secure private randomness generated inside the enclave without an external oracle
- Selective disclosure via programmable decryption permissions per encrypted value

## Enterprise demand and use cases

- Confidential token standard work with Circle Research, OpenZeppelin, and Zama (Confidential Token Association)
- Confidential ERC-3643 for RWA issuance with private, enforceable compliance logic
- Private payroll, vesting, and treasury flows where amounts must be hidden but counterparties are known and screenable
- Private DeFi primitives: blind auctions, confidential money markets, DvP escrow, private governance
- Prediction markets requiring hidden state and provable fairness

## Technical details

- Standard Solidity, Foundry/Hardhat tooling, npm-distributed library (`@inco/lightning`). No new VM or language.
- Runs at the speed of the host chain with near-zero added latency. No bridging to a separate privacy chain.
- Composable: confidential contracts interoperate with each other and with public DeFi on the same chain.
- Local development environment via Docker for testing without live enclaves.

## Strengths

- Lowest integration cost in the confidential-compute category: one import into existing Solidity contracts
- Performance suitable for latency-sensitive applications (real-time games, high-frequency flows) where zero-knowledge or Fully Homomorphic Encryption approaches are currently impractical
- Post-quantum-secure symmetric encryption of state
- Users keep existing wallets and the host chain's settlement guarantees
- Verifiable compute end to end

## Risks and open questions

- Hardware trust anchors: enclave vendor compromise or microarchitectural side channels are outside the cryptographic model
- Liveness depends on the coprocessor.
- The confidentiality model (values hidden, addresses public) leaks the transaction graph. 

## Links
- [Website](https://www.inco.org/)
- [Documentation](https://docs.inco.org)
- [GitHub](https://github.com/Inco-fhevm)
- [Circle Research x Inco: Confidential ERC20 Framework](https://github.com/Inco-fhevm/confidential-erc20-framework/blob/main/whitepaper.pdf)
