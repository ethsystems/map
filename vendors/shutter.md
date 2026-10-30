---
title: "Vendor: Shutter"
status: ready
maturity: production
---

# Shutter Network – Encrypted Mempool & MEV Protection

## What it is

Shutter Network provides encrypted-mempool infrastructure that prevents MEV extraction and front-running by encrypting transactions until their ordering is committed, using threshold cryptography for decryption. Brainbot develops it. Shutterized Gnosis Chain has been live since July 2024, opt-in through a Shutter RPC. On Ethereum mainnet no Shutter encrypted mempool is live as of October 2026.

## Fits with patterns

- [Threshold-Encrypted Mempool](../patterns/pattern-threshold-encrypted-mempool.md): Shutter is the reference implementation
- [Pre-trade Privacy](../patterns/pattern-pretrade-privacy-encryption.md): encrypted mempool with threshold decryption
- [Private Broadcasting](../approaches/approach-private-broadcasting.md): the Encrypted Mempools sub-approach
- [Publicly Verifiable DKG & Threshold Decryption](../patterns/pattern-verifiable-dkg-threshold-decryption.md): the keyper committee's key generation and decryption layer

## Not a substitute for

- Post-execution privacy (transactions are visible after inclusion)
- Cross-chain MEV protection
- Complex intent expression (focuses on transaction-level protection)

## Architecture

A committee of keypers runs a distributed key generation and publishes a shared public key. Users encrypt a transaction to that key for an identity, such as a slot. Once the ordering of the encrypted transactions is committed, at least a threshold of keypers release shares of the decryption key for that identity, and the transactions execute in the committed order. Searchers and builders cannot read content during the mempool phase.

## Privacy domains

- **Encrypted mempool (Gnosis Chain):** transactions sent through a Shutterized RPC stay encrypted until the keypers release the slot's key.
- **Shutter API:** a commit-and-reveal (timelock) encryption service on Gnosis Chain and Chiado, with its own keyper sets, for applications such as sealed-bid auctions and private voting.

## Enterprise demand and use cases

- Encrypted mempool preventing MEV extraction during broadcasting
- Threshold decryption with no single point of failure
- Sealed-bid and timelocked submissions through the Shutter API

## Technical details

- **Networks:** Gnosis Chain mainnet (encrypted mempool, 7 keypers with a threshold of 4); Shutter API on Gnosis Chain and Chiado
- **Ethereum:** an out-of-protocol proof of concept with Primev's mev-commit was announced for the Hoodi testnet in December 2025 (3-of-5 keypers, builders commit before keys are released). The mainnet launch had not happened as of September 2026, and Brainbot's [September 2026 update](https://shutternetwork.discourse.group/t/976) says the out-of-protocol work is blocked by Primev resource constraints.
- **Enshrinement:** Shutter authored [EIP-8105](https://eips.ethereum.org/EIPS/eip-8105) (Draft), an enshrined encrypted mempool. Its bid to headline the Hegotá upgrade was withdrawn in February 2026 in favour of LUCID ([EIP-8184](https://eips.ethereum.org/EIPS/eip-8184)), which the team now supports. The same update sets the goal of an encrypted mempool EIP ready to propose as headliner for I*, the upgrade after Hegotá.
- **Code:** [rolling-shutter](https://github.com/shutter-network/rolling-shutter) (Go, MIT) is the active keyper implementation; the original `shutter` repository is legacy.
- **Compatibility:** works with standard Ethereum transactions

## Strengths

- Production deployment on Gnosis Chain since 2024
- Threshold decryption: no single keyper can read pending transactions
- Open-source keyper software
- Active in the Ethereum protocol process for an enshrined encrypted mempool

## Risks and open questions

- No production deployment on Ethereum mainnet
- On Gnosis Chain, users trust the Shutter RPC that receives their transaction, and validators are not slashed for misbehaviour
- A transaction re-sent after its key was released can be front-run
- Adds latency due to the key-release step
- Requires coordination among threshold key holders
- No protection against MEV after transaction execution

## Links

- [Shutter Network](https://shutter.network/)
- [Documentation](https://docs.shutter.network/)
- [GitHub](https://github.com/shutter-network)
- [Shutterized Gnosis Chain launch (July 2024)](https://shutternetwork.discourse.group/t/shutterized-gnosis-chain-debuts-in-july/444)
- [Threshold-encrypted mempools with mev-commit preconfirmations](https://ethresear.ch/t/threshold-encrypted-mempools-with-mev-commit-preconfirmations/23588)
- [Shutter API](https://github.com/shutter-network/shutter-api)
