---
title: "Vendor: Gateway Open Privacy Suite"
status: ready
website: https://gateway.fm/open-privacy-suite/
category: privacy-layer
ethereum_aligned: true
last_reviewed: 2026-08-07
maturity: testnet
---

# Gateway.fm - Open Privacy Suite (governed access for EVM networks)

## What it is

Gateway Open Privacy Suite (OPS) is an Apache-2.0, self-hostable access and
disclosure layer for shared EVM networks. It sits in front of standard JSON-RPC
nodes and applies identity, role-based access control (RBAC), response filtering,
selective disclosure, compliance rules, and audit logging without changing the
underlying smart contracts.

OPS can govern an existing private EVM, including a Besu permissioned network,
or be composed with a Gateway-deployed L2 or validium that keeps transaction data
off the public chain and posts proofs or commitments for settlement. Gateway
describes the suite as testnet ready. OPS itself controls access to state; it
does not encrypt the underlying ledger.

## Fits with patterns

- [Privacy L2s](../patterns/pattern-privacy-l2s.md)
- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md)
- [Shielding](../patterns/pattern-shielding.md)
- [Modular Privacy Stack](../patterns/pattern-modular-privacy-stack.md)
- [Selective Disclosure (Viewing Keys + Zero-Knowledge Proofs)](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Compliance Monitoring](../patterns/pattern-compliance-monitoring.md)
- [zk-KYC/ML + ERC-734/735 identity claims](../patterns/pattern-zk-kyc-ml-id-erc734-735.md)

The first three patterns apply when OPS is paired with a private L2 or validium;
the remaining patterns describe its access, identity, disclosure, and compliance
control plane.

## Not a substitute for

- Protocol-level private execution: in the baseline configuration, validators,
  node operators, and anyone who bypasses the proxy can read the ledger.
- Data-availability and bridge security for a private L2 or validium.
- Organizational controls for administrator access, key custody, backups, and
  incident response.

## Architecture

Clients authenticate through Privado ID zero-knowledge credentials, Microsoft
Entra ID, or linked wallets and continue to use Ethereum JSON-RPC. OPS resolves
organization, group, user, method, contract, function, and parameter permissions.
Before forwarding a transaction, it can use `debug_traceCall` to inspect internal
calls and reject access through intermediary or multicall contracts.

Responses and explorer views are filtered for the caller. Disclosure grants can
be scoped, consented, time-limited, and logged. PostgreSQL stores policies and
RBAC data; a separate append-only database stores the audit trail; Redis is
optional for multi-instance sessions and shared policy state. The privacy
topology places every node and indexer behind controlled interfaces because one
unprotected node exposes the raw ledger.

Deployment profiles include:

- **Private-chain access layer:** OPS in front of a compatible EVM execution
  client, including Geth, Erigon, Besu, and Nethermind. This also applies to
  permissioned Besu/QBFT deployments.
- **Private L2 or validium:** OPS combined with Gateway Presto and an EVM rollup
  stack such as cdk-erigon, with transaction data retained outside the public
  settlement chain and ZK proofs or commitments posted to Ethereum.
- **Encrypted L2 state add-on:** an optional state-encryption profile for
  Reth-based deployments. It is not available for other execution clients.

## Privacy domains

- Tenant and user isolation at the JSON-RPC, contract, function, parameter, and
  explorer layers.
- Full, pseudonymous, or redacted disclosure to auditors, regulators, and other
  approved parties.
- Off-public-chain transaction data when deployed as a private L2 or validium.
- Optional encrypted node state for the Reth-specific add-on.

## Enterprise demand and use cases

- Private stablecoin, tokenized-deposit, wholesale-payment, and cross-border
  settlement networks with KYC, travel-rule, and auditor access requirements.
- Bilateral trading, private order flow, delivery-versus-payment, repo, FX, and
  capital-markets settlement on a shared EVM.
- Confidential issuance and lifecycle management for funds, bonds, treasuries,
  commodities, and other real-world assets.
- Consortium and regulated application infrastructure that requires ordinary
  Solidity tooling, tenant-specific views, and evidence of policy enforcement.

## Technical details

- Go proxy and APIs, React administration UI, PostgreSQL policy stores, and an
  optional Redis layer.
- Standard Ethereum JSON-RPC, so Solidity contracts and tools such as Foundry,
  Hardhat, ethers.js, and web3.js do not require a different execution model.
- Runtime tracing, multicall detection, method allowlists, contract grants,
  function/parameter rules, KYC gates, and travel-rule checks.
- Privacy-filtered explorer and indexer, with append-only, hash-chained access
  records for later verification.

## Strengths

- Execution-client portability separates the policy layer from the EVM beneath
  it and permits use with existing private Besu networks.
- Identity, authorization, disclosure, compliance, explorer views, and audit
  evidence use the same policy boundary.
- Open-source implementation can be self-hosted and inspected under Apache-2.0.
- L2/validium composition covers the same institutional use-case categories as
  a purpose-built privacy L2 while retaining a reusable EVM access layer.

## Risks and open questions

- The proxy operator and raw node operators remain trusted in the baseline OPS
  model. A direct node, indexer, backup, or administrative bypass defeats
  access-layer confidentiality.
- Filtering correctness depends on the execution client's tracing behavior,
  complete coverage of JSON-RPC and explorer surfaces, and fail-closed policy.
- L2/validium deployments add sequencer, bridge, proving, data-availability,
  recovery, and upgrade-key assumptions that must be evaluated separately.
- The Reth state-encryption add-on introduces encryption-key custody, rotation,
  recovery, snapshot, and runtime plaintext-handling requirements.
- Testnet readiness does not by itself establish production performance,
  resilience, audit coverage, or security review for a specific deployment.

## Links

- [Open Privacy Suite](https://gateway.fm/open-privacy-suite/)
- [Open Privacy Suite documentation](https://gateway-fm.github.io/open-privacy-suite/)
- [What it does and trust model](https://gateway-fm.github.io/open-privacy-suite/docs/what-it-does/)
- [Architecture](https://gateway-fm.github.io/open-privacy-suite/docs/architecture/)
- [Security model](https://gateway-fm.github.io/open-privacy-suite/docs/security/)
- [Selective disclosure](https://gateway-fm.github.io/open-privacy-suite/docs/disclosure/)
- [Source code](https://github.com/gateway-fm/open-privacy-suite)
- [Gateway Presto](https://gateway.fm/presto/)
- [cdk-erigon validium configuration](https://docs.gateway.fm/cdk-erigon/configuration-options/)
