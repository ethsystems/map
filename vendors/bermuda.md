---
title: "Vendor: Bermuda"
status: ready
website: https://bermudabay.xyz/
category: privacy-layer
ethereum_aligned: true
last_reviewed: 2026-09-03
maturity: testnet
---

# Bermuda (Privacy and compliance layer for the EVM)

## What it is

Bermuda is a privacy and compliance layer deployed as contracts on existing
EVM chains. Funds sit in a smart contract as shielded UTXOs: balances, amounts and
counterparties are hidden by zero-knowledge proofs generated on the user's
device, and every transfer proves in zero knowledge that it satisfies the
issuer's policy for that token. Sanctions screening, retroactive flagging with
Proof of Innocence, issuer policies and clawback are enforced by the smart
contract and its circuits, not by an operator who reads transactions. It is
middleware, not a chain, wallet or custodian.

Compliance is a pluggable module rather than a fixed rule set. An
integrator configures KYC/AML workflows, jurisdiction-specific rules and
custom policies without modifying the core protocol. Deposit screening is
delegated to an external compliance engine that the deployment chooses.
Two deployments under different regulatory regimes can therefore share one
privacy layer.

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

- **Smart contract**: holds funds as shielded UTXOs. Commitments sit in an
  incremental Merkle tree. Each spend publishes a nullifier bound to that
  chain. Notes are encrypted to the recipient and emitted as events.
- **Proofs**: circuits are written in Noir. Proving runs on the user's own
  device.
- **Issuer policy engine**: the token owner, typically a Safe, sets a
  per-token policy covering a spending limit per period, allow and deny lists
  and an optional viewing key. Every transfer proves in-circuit that it
  satisfies the live policy. The circuit checks a commitment to the policy
  fields, so an issuer can publish its rules or keep them private. List
  membership stays off-chain in either case. The viewing key decrypts that
  token's transfers. The issuer can claw back funds with a proof.
- **Compliance gateway**: a deposit binds the compliance engine's approval and
  an expiry block into the proof. A delisted address cannot replay an old
  approval. Flagged deposit ids are tracked in an indexed Merkle tree. A
  private withdrawal proves the deposit is absent from that tree. A disclosed
  withdrawal proves it is present. Depositors and recipients are also screened
  on-chain by the Chainalysis sanctions oracle.
- **Submission path**: a Bermuda relayer submits shielded transactions on
  behalf of users. ERC-4337 bundlers and an x402 facilitator cover gas
  abstraction and agent payments.
- **Roles**: the smart contract and its verifiers are non-upgradeable. A
  governor sets fees, recovery parameters and the public-withdrawal delay. The
  governor assigns a pauser who halts `transact` and `claim` for incident
  response.
- **Cross-chain DvP**: two contracts act as escrows. One leg locks, then
  releases once the counterparty's lock is proven present. It is reclaimed on
  a proof of absence after the cutoff. No coordinator sits in the path. Linea
  proof of concept, June 2026.
- **Products**: SDK suite (`core-sdk`, `issuer-sdk`, `safe-sdk`,
  `fireblocks-sdk`) and end-user apps (Private Safe Wallet, with universal and
  mobile apps in progress).
- **Services**: enterprise delivery from build through operations to
  compliance. Private OTC, DvP and RFQ networks. Institutional wallet UX.
  Accounting, reporting and compliance adapters. MPC overlay on institutional
  HSMs through the Utila partnership.

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

Related approaches: [Private Payments](../approaches/approach-private-payments.md),
[Atomic DvP Settlement](../approaches/approach-dvp-atomic-settlement.md),
[Private Trade Settlement](../approaches/approach-private-trade-settlement.md),
[White-Label Infrastructure Deployment](../approaches/approach-white-label-deployment.md)

Related use cases: [Private Stablecoins](../use-cases/private-stablecoins.md),
[Private RWA Tokenization](../use-cases/private-rwa-tokenization.md),
[Private Treasuries](../use-cases/private-treasuries.md),
[Private Payments](../use-cases/private-payments.md),
[Private Repo](../use-cases/private-repo.md),
[Private Stocks](../use-cases/private-stocks.md)

## Technical details

- Proving curve: BN254. Grumpkin is its cycle partner and carries the spend
  signatures
- Proof system: Noir with Barretenberg (UltraHonk). Universal SRS, so there
  is no circuit-specific trusted setup
- Spend authorization: Schnorr over Grumpkin. FROST supplies the threshold
  variant, keyed by Golden DKG
- Note key agreement: X25519 ECDH, with a fresh ephemeral keypair per note
- Note encryption: XChaCha20-Poly1305 under the derived shared secret

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
- Relayer dependence: the Bermuda relayer sees submission metadata and can
  censor or delay a shielded transaction. Self-submission is the fallback,
  at the cost of network-level anonymity
- Compliance engine dependence: entry to the pool requires an approval from
  an external screening provider
- Governor powers: fees, recovery parameters and the public-withdrawal delay
  are governor-controlled. The pauser can halt transfers and claims
- Issuer powers cut both ways: freezing and clawback follow holders into the
  shielded domain
- Client-side proving cost on constrained devices is not published

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
