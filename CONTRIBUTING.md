# Contributing to EthSystems Map

## How to Contribute

### Adding a Vendor/Protocol

1. Check the [vendors directory](./vendors/) - don't duplicate existing ones
2. Create a new file: `vendors/your-protocol.md`
3. Use the [template](./vendors/_template.md)
4. Submit a Pull Request using the vendor template

### Adding an Enterprise Use Case

1. Check the [use-cases](./use-cases/) and [approaches](./approaches/) directories before starting
2. Create a new file describing your use case
3. Submit a Pull Request using the use-case [template](./use-cases/_template.md)

## Simple Rules

- **Be accurate** - no marketing, just facts
- **Check for duplicates** - don't add what's already there
- **Include working links** - make sure documentation works
- **Use semantic commits** - helps us track changes

**Semantic commit types:**

- `feat:` - New vendor, pattern, use case, or approach
- `fix:` - Corrections to existing content
- `docs:` - Documentation updates
- `refactor:` - Reorganizing content without changing meaning
- `chore:` - Maintenance tasks

## Changelog

When adding new patterns, vendors, or significant changes:

1. Add an entry to `[Unreleased]` section in [CHANGELOG.md](./CHANGELOG.md)
2. Include a markdown link to the new file
3. Reference the PR number: `(#123)`

Example entry:

```markdown
- [New pattern name](patterns/pattern-slug.md) (#123)
```

## CROPS Evaluation

CROPS are the four non-negotiable properties defined by the Ethereum Foundation. They are indivisible: a solution that satisfies three out of four is not CROPS-aligned.

| Letter | Property                  | Definition                                                                        |
| ------ | ------------------------- | --------------------------------------------------------------------------------- |
| **CR** | **Censorship Resistance** | No actor can selectively exclude valid use or break functionality                 |
| **O**  | **Open Source and Free**  | No privileged code or hidden specifications; forkable with predictable exit paths |
| **P**  | **Privacy**               | User data is not exposed beyond necessity or against their interests              |
| **S**  | **Security**              | Things must do what they claim to do, no more and no less                         |

### Censorship Resistance (CR)

| Score    | Meaning                                                                                                                                  |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `high`   | Permissionless access; no single party can exclude a user; inclusion or exit is guaranteed at the protocol level                         |
| `medium` | Access is gated, but users retain a credible independent path to participate or exit without institutional approval                      |
| `low`    | Access is gated and exclusion is feasible in practice; user exits exist but are weak, delayed, or operationally constrained              |
| `none`   | The system itself is the censorship vector: freeze, blacklist, approval gate, revocation power, or admin control over user participation |

In I2U contexts, `medium` requires a concrete user escape path such as forced withdrawal, credential portability, or an L1 exit. Without that, the institution is the effective point of control over user participation.

If the answer to both questions below is “yes” and the fallback is not independently enforceable, the score should usually be `low` or `none`.

Use these to justify the score in one or two lines:

- Can one identifiable actor stop the user from participating?
- Can one identifiable actor stop the user from exiting?
- Is there a non-custodial or protocol-enforced fallback?
- Does the fallback work without institutional/operator approval?
- Is exclusion exceptional and transparent, or built into normal operation?

### Open Source and Free (O)

| Score     | Meaning                                                                                                                                                 |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `yes`     | Fully open source, publicly auditable, and forkable; predictable exit paths; no proprietary black boxes in the critical path                            |
| `partial` | Core logic is open; some components (prover, oracle, bridge) are proprietary or source-available but not forkable; exit paths exist but are constrained |
| `no`      | Closed source or significant proprietary components in the critical path; no credible exit or fork path                                                 |

"Open Source and Free" goes beyond source availability. It requires that systems are forkable with credible exit paths — users and builders must be able to leave without permission. Requires an explicit open license: copyleft (GPL, AGPL) is preferred, permissive (MIT, Apache 2.0, CC0) is accepted, source-available or proprietary licenses do not qualify. Projects should commit to not relicensing away from open source or copyleft in the future.

In I2U contexts, the tools provided by institutions to users must be public and auditable, with open specifications. Source availability alone does not count if the institution deploys opaque modifications and users have no way to check.

### Privacy (P)

| Score     | Meaning                                                                                                                                                                                       |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `full`    | Privacy by default. No party learns more than the protocol requires. Users don't need to prove anything or opt in.                                                                            |
| `partial` | Private by default, but structured disclosure is required for specific parties (e.g., regulator audit, counterparty identity). Disclosure is scoped and always downstream of default privacy. |
| `none`    | Data is public on-chain, or the operator sees all user operations.                                                                                                                            |

Examples:

- Shielded pool, no party can observe user activity → `full`
- Shielded pool with viewing keys where the user controls disclosure → `full` (viewing keys are a tool, not a disclosure requirement)
- Users must disclose activity to a regulator or counterparty to participate → `partial`
- I2I trade: amounts and asset hidden, counterparty identities known → `partial`
- I2U system where the operator sees all user operations → `none`

### Security (S)

| Score    | Meaning                                                                                                                                                              |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `high`   | Well-established cryptographic assumptions; no single point of failure; no privileged upgrade path; passes the walkaway test (keeps working if the team disappears)  |
| `medium` | Security relies on operational trust assumptions (e.g., honest-majority MPC, TEE attestation, trusted setup) or has upgrade risk; depends on active team maintenance |
| `low`    | Weak or deprecated cryptographic assumptions; centralized trust; significant upgrade or key management risk; high operational complexity that concentrates risk      |

Complexity is a risk in itself. Prefer simple, verifiable designs over elaborate ones. Document specific assumptions (e.g., discrete log hardness, SGX attestation trust, honest-majority threshold) in the pattern body under Trade-offs.

For **vendors/products**: the score combines the cryptographic assumptions of the underlying primitives with operational factors — formal audit status, upgrade key controls, admin key scope, incident history, and bug bounty program.

In I2U contexts, security also covers what the institution cannot do to the user unilaterally — not just what the system does correctly.

## Getting Help

- Open an issue for questions
- Use GitHub Discussions for broader topics
