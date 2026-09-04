---
title: "Vendor: Zama"
status: ready
maturity: mainnet
---

# Zama – Confidential Blockchain Protocol (Fully Homomorphic Encryption on existing chains)

## What it is

Zama develops open-source Fully Homomorphic Encryption (FHE) tooling and operates the Zama Confidential Blockchain Protocol, which lets smart contracts compute on encrypted data on existing public chains. Zama describes it as "not a new L1 or L2, but rather a cross-chain confidentiality layer sitting on top of existing chains", so applications stay on their host chain instead of migrating to a dedicated network. Ethereum mainnet is live; further EVM chains and Solana are on the published roadmap.

## Fits with patterns

- [Private Shared State (FHE)](../patterns/pattern-private-shared-state-fhe.md) - encrypted state shared between parties, computed off-chain by coprocessors
- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md) - encrypted balances and transfer amounts
- [Private Intent-Based Vaults](../patterns/pattern-private-vaults.md) - strategy parameters stay encrypted during execution
- [Shielding](../patterns/pattern-shielding.md) - partial fit: ERC-7984 hides amounts and balances, but sender and receiver stay public

## Not a substitute for

- Graph privacy: sender and recipient addresses remain public, so unlinkability needs composition with stealth addresses
- Privacy rollups that move execution and state off the host chain entirely
- Multi-party computation or Trusted Execution Environments as general-purpose off-chain compute

## Architecture

Contracts import the FHEVM Solidity library and use encrypted types (`euint8` to `euint256`, `ebool`, `eaddress`). Each encrypted value is referenced on-chain by a 32-byte handle. Zama's documentation places the ciphertext itself "off-chain, with the coprocessor" and the handle on-chain as the identifier pointing to it. FHE operations run symbolically on the host chain, generating new handles and emitting events, while a network of coprocessors performs the computation using TFHE-rs.

The protocol combines the FHEVM Solidity library; host contracts that enforce access control and emit events; coprocessors that verify inputs, execute FHE operations and store ciphertexts; the Gateway, a dedicated Arbitrum rollup that orchestrates input validation, decryption and bridging; and a threshold multi-party computation Key Management Service. Relayer and oracle services connect users and contracts to the Gateway and are not part of the trusted base.

## Privacy domains

- Fully Homomorphic Encryption at the contract and variable level: amounts and balances stay encrypted end to end
- Programmable access per ciphertext through an Access Control List (`allow`, `allowThis`, `allowTransient`, `makePubliclyDecryptable`), including delegation of user decryption
- Hybrid by design: zero-knowledge proofs validate encrypted inputs, multi-party computation performs threshold decryption

## Enterprise demand and use cases

- Confidential payments and stablecoin transfers where amounts and balances stay hidden from other participants
- Tokenization and real-world assets where holdings and allocation sizes are confidential but settlement stays on a public chain
- Sealed-bid auctions, confidential distributions and governance, where inputs stay private until a defined reveal

## Technical details

- FHEVM Solidity library (`@fhevm/solidity`), with a relayer SDK and a Hardhat plugin
- Confidential tokens follow ERC-7984, implemented in OpenZeppelin's confidential-contracts library
- Encrypted inputs carry a zero-knowledge proof of knowledge, validated through `FHE.fromExternal`
- Conditional logic uses `FHE.select` instead of plaintext branching, so the taken branch is not revealed
- Protocol fees apply to input verification, decryption and bridging. They are priced in USD and can be paid by the end user, the application or a relayer

## Strengths

- Host contracts deploy onto existing chains, EVM or non-EVM, so users do not move to a dedicated network
- Composable between confidential contracts and with non-confidential ones, and contracts are written in plain Solidity rather than a new language
- Every FHE computation and input verification carries a commitment and a signature, so anyone can recompute the result and check it independently
- No single party ever holds the decryption key: it is secret-shared across the threshold network, and each decryption is authorized by the Access Control List
- Compliance rules are defined by each application in its own contracts rather than by the protocol
- Audited by Trail of Bits and Zenith, and the FHE and multi-party computation layers are post-quantum

## Risks and open questions

- The trust model is layered: decryption runs through a threshold Key Management Service of 13 multi-party computation nodes, with a documented threshold example of 9 of 13 and tolerance up to one third malicious; those nodes run inside AWS Nitro Enclaves, so verifiability rests partly on hardware assumptions; and coprocessor correctness rests on a majority-honest assumption backed by staking and slashing. Zama states the goal is to add zero-knowledge proofs to the multi-party computation protocol to remove the hardware dependency
- Operators can pause the protocol and blacklist addresses, and the Access Control List carries an account deny list
- Open source under a dual license: free for non-commercial use, with a commercial license required for use outside the Zama Protocol
- Cost and latency stay above plaintext execution, and protocol support beyond Ethereum mainnet is roadmap rather than shipped

## Links

- [Zama Protocol documentation](https://docs.zama.org/protocol)
- [Litepaper](https://docs.zama.org/protocol/zama-protocol-litepaper)
- [Handles: what lives on-chain and off-chain](https://docs.zama.org/protocol/solidity-guides/smart-contract/handles)
- [Coprocessor](https://docs.zama.org/protocol/protocol/overview/coprocessor)
- [Gateway](https://docs.zama.org/protocol/protocol/overview/gateway)
- [Key Management Service](https://docs.zama.org/protocol/protocol/overview/kms)
- [Access Control List](https://docs.zama.org/protocol/solidity-guides/smart-contract/acl)
- [zama-ai/fhevm](https://github.com/zama-ai/fhevm)
- [TFHE-rs](https://github.com/zama-ai/tfhe-rs)
