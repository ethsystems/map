---
title: "Domain: Custody"
status: draft
description: "Protect keys, controls, and recovery while preserving scoped oversight for legitimate auditors and successors."
---

## TLDR
- Wallet ops/KMS/HSM, freeze/blacklist controls, segregation, recovery, post-trade hooks.
- Governed regulator access (view keys / proofs) and evidence logging.

## Primary use cases
- (Cross-cut; referenced by Payments and Funds & Assets flows)

## Shortest-path patterns
- [Selective disclosure (view keys + proofs)](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [Atomic DvP via ERC-7573](../patterns/pattern-dvp-erc7573.md)

## Adjacent vendors
- [Kaleido Paladin](../vendors/paladin.md)
- [Fireblocks](../vendors/fireblocks.md)
- [iExec](../vendors/iexec.md)
- [Soda Labs](../vendors/soda-labs.md)
- [Inco](../vendors/inco.md)