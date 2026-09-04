---
title: "Vendor: Bermuda"
status: ready
website: https://bermudabay.xyz/
category: privacy-layer
ethereum_aligned: true
last_reviewed: 2026-09-03
maturity: testnet
---

# Bermuda – Privacy and compliance layer for the EVM

## What it is

Bermuda is a privacy and compliance layer deployed as contracts on existing
EVM chains. Funds sit in a smart contract as shielded UTXOs: balances, amounts and
counterparties are hidden by zero-knowledge proofs generated on the user's
device, and every transfer proves in zero knowledge that it satisfies the
issuer's policy for that token. Sanctions screening, retroactive flagging with
Proof of Innocence, issuer policies and clawback are enforced by the smart
contract and its circuits, not by an operator who reads transactions. It is
middleware, not a chain, wallet or custodian.

## Fits with patterns

- [Shielding](../patterns/pattern-shielding.md)
- [Proof of Innocence](../patterns/pattern-proof-of-innocence.md)
- [Compliance Monitoring](../patterns/pattern-compliance-monitoring.md)
- [Regulatory Disclosure Keys & Proofs](../patterns/pattern-regulatory-disclosure-keys-proofs.md) - issuer viewing keys
- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md)
- [Stealth Addresses](../patterns/pattern-stealth-addresses.md)
- [Private Transaction Broadcasting](../patterns/pattern-private-transaction-broadcasting.md)
- [Immutable Guarantees](../patterns/pattern-immutable-guarantees.md)
- [Social Recovery](../patterns/pattern-social-recovery.md)
- [DvP (ERC-7573)](../patterns/pattern-dvp-erc7573.md) - partial fit: cross-chain DvP via ERC-7888 proofs

## Not a substitute for

- Not a privacy L2 or separate ledger: everything runs on the host chain; no
  sequencer, coprocessor network or bridge in the transaction path
- Not a wallet or custodian: users keep their keys in existing wallets

## Architecture

- **Smart contract**: holds funds as shielded UTXOs: commitments in
  an incremental Merkle tree, chain-bound nullifiers, one `transact` entry point.
  Notes are encrypted to the recipient's x25519 key and emitted as events.
- **Proofs and keys**: Noir circuits, Barretenberg as ZK proving backend, 
  client-side proving in WASM. Spend authorization is Schnorr over Grumpkin; FROST
  threshold Schnorr puts a Safe's owners behind a shielded account.
- **Issuer policy engine**: the token owner (typically a Safe) sets a
  per-token policy — spending limit per period, allow and deny lists, an
  optional issuer viewing key — that every transfer proves in-circuit against
  the smart contract's live policy root. The circuit checks a commitment to the policy's
  fields, so an issuer can publish its rules or keep them private; list
  membership stays off-chain either way. The viewing key yields an encrypted
  view of that token's transfers; the issuer can claw back funds with a proof.
- **Compliance Gateway**: deposits bind the compliance engine's key and an
  expiry block into the proof, so a delisted address cannot replay an
  approval. Flagged deposit ids live in an indexed Merkle tree: private
  withdrawals prove exclusion, disclosed ones inclusion. Depositors and
  recipients are also screened on-chain by the Chainalysis sanctions oracle.
- **Roles**: the smart contract and verifiers are non-upgradeable. A governor
  sets fees,
  recovery parameters and the public-withdrawal delay and assigns a pauser,
  who halts `transact` and `claim` for incident response.
- **Cross-chain DvP**: two smart contracts act as escrows: a leg is locked,
  released
  once the counterparty's lock is proven present, or reclaimed on a proof of
  absence after the cutoff — ERC-7888 broadcast proofs, no coordinator. Linea
  proof of concept, June 2026.
- **Products**:
  - SDK suite: `core-sdk`, `issuer-sdk`, `safe-sdk`, `fireblocks-sdk`, etc.
  - E2E apps: Private Safe Wallet, universal/mobile app in progress
- **Services**:
  - E2E enterprise applications, from development through operations to compliance, for instance:
    - Exclusive OTC/DvP/RFQ networks (always private, cross-chain capable)
    - Customized private wallet UX, especially enterprise- and institution-focused
    - Accounting/reporting and compliance adapters, and middleware
  - MPC overlay upgrades on top of institutions’ physical HSMs through the partnership with Utila

## Privacy domains

- Balances, amounts and the transaction graph are hidden. Stealth addresses
  keep public 3rd party protocol interactions unlinkable.
- Two exits: private (exclusion proof, deposit not revealed) and disclosed (if flagged by the compliance engine).
- Disclosure is scoped: an issuer viewing key decrypts transfers of that
  issuer's token and nothing else; the compliance engine sees the depositor's
  public address and never activity inside the contracts.

## Enterprise demand and use cases

- **Stablecoin, deposit-token and RWA issuers**: private or public policies,
  issuer viewing key and clawback give the issuer the controls expected of a
  regulated token while holders keep confidential balances
- **Regulated tokenised assets**: the privacy example (Box 5) in the Global
  Layer One white paper *Programmable Compliance* (June 2026, with Banque de
  France, the IMF, Kinexys by J.P. Morgan, MAS and Standard Chartered)
- **Banks and institutions**: shielded treasury under Safe multisig,
  recoverable accounts, atomic cross-chain DvP and PvP, bilateral repo and OTC
  swaps
- **Wallets and neobanks**: shielded balances and transfers behind existing UX
- **Payments**: payroll, card settlement, x402
- **Asset management**: shielded ERC-4626 positions, private yield and DeFi,
  private order books

## Technical details

- Curves in use: BN254 (proof system), Grumpkin (signatures), Curve25519 (encryption)
- Proof system: Noir, Barretenberg (UltraHonk); no circuit-specific trusted setup (universal SRS)
- Signing: Schnorr, FROST, Golden DKG
- Encryption: XChaCha20-Poly1305

Post-quantum implementation in progress.

## Strengths

- Issuer self-onboarding: any token owner can attach and update a policy
  without Bermuda's involvement
- Signer-agnostic: multisig, passkey and MPC signers, plus account recovery
- Compliance is proved, not observed: screening, flagging, sanctions, issuer
  policy and Proof of Innocence are verified by contract and circuit
- Issuer powers are strong by design: an issuer keeps the abilities it has on
  the public chain, selective freezing and clawback included, in the shielded domain too
- Trust model is the host chain plus standard zk-SNARK assumptions: no Trusted
  Execution Environment, multi-party computation committee or coprocessor
  network in the confidentiality path
- No new ledger: same chain, same tokens, same wallets
- Exit is protocol-level: withdrawal is always available, private for clean
  and disclosed for flagged lineage; the operator cannot freeze an account

## Risks and open questions

- Testnet stage: no mainnet deployment, third-party audit or bug bounty published
- Anonymity depends on activity in the contracts per chain and asset

## Links

- [Bermuda website](https://bermudabay.xyz/)
- [Documentation](https://docs.bermudabay.xyz/)
- [Architecture](https://docs.bermudabay.xyz/tech/architecture)
- [Contracts](https://docs.bermudabay.xyz/tech/contracts)
- [Compliance overview](https://docs.bermudabay.xyz/compliance-solution/how-compliance-works)
- [GitHub organization](https://github.com/BermudaBay)
- [GL1 white paper: Programmable Compliance, Box 5](https://global-layer-one.org/pdf/gl1-pc-whitepaper-22-jun-2026.pdf)
- [Linea × Bermuda settlement PoC](https://cryptobriefing.com/linea-bermuda-private-cross-chain-settlement/)
- [Utila x Bermuda partnership](https://utila.io/blog/utila-bermuda-confidential-custody-central-banks)
