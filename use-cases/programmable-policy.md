

| title | primary\_domain | secondary\_domain |
| :---- | :---- | :---- |
| Compliant privacy via programmable policy | Identity & Compliance |  |

## 1\) Use case

Institutional users need privacy tooling to keep competitive information (trading strategies, portfolio composition, risk exposure, vendor payments, etc.) confidential when transacting onchain. But the same privacy mechanisms that protect legitimate institutional activity can also be leveraged by malicious actors, creating business, legal, and regulatory risks.

As a result, institutions using privacy-enhanced protocols and chains (e.g. private vaults, shielded transfer systems, privacy L2s), or issuing stablecoins and other RWA tokens that may circulate through such environments, need tools that allow them to manage business and regulatory risk. More specifically, they need tools enabling them to avoid transacting with or commingling funds with high-risk entities, even when privacy features are in use.

## 2\) Additional Business Context



## 3\) Actors

Issuer, Investor, Payer/Payee, Compliance/Monitoring, Law enforcement, Regulator, Privacy protocol operator

## 4\) Problems

### Problem 1: Preventing commingling of funds with malicious actors when using privacy protocols

Institutions may use DeFi protocols with privacy enhancements (e.g. [private vaults](https://github.com/ethereum/iptf-map/blob/master/patterns/pattern-private-vaults.md)) to complete onchain investments, payments, and other financial operations without exposing their strategy or risk exposure. This can be done using encryption protocols on Ethereum mainnet or protocols on privacy L2s. However, privacy enhancements can make it harder for institutions to avoid commingling of funds with deposits from malicious actors, as is mandated by regulations and/or internal risk management requirements.

**Requirements:**

- Must allow protocol operators to prevent commingling of funds and deny deposits from malicious actors at the smart contract level while still preserving privacy

**Constraints:**

- Any solution must not compromise privacy or introduce significant latency or UX detractions for compliant users

## 5\) Recommended approaches

Operators of privacy protocols and privacy L2s can set programmable policies restricting high-risk or malicious actors from using their tools – those policies are then integrated and enforced at the smart contract level. 

For example, a privacy project operator may decide to implement a policy restricting all addresses on the OFAC sanctions list and addresses with exposure to certain high-risk activity (e.g. terrorism financing, darknet market sales, etc.) from depositing funds to their project.

If the project in question is a privacy-enhanced application on Ethereum mainnet, the application team can integrate this policy into its smart contracts and automatically prevent restricted addresses from depositing funds. This provides more consistent compliance enforcement than the common framework of front-end compliance checks, which can be bypassed via direct contract calls. If the project is a privacy L2, the operator can integrate their programmable policy with the bridge contract, and prevent restricted addresses from moving funds to the chain at all. 

In both cases, the application calls the policy platform, which returns a cryptographic attestation confirming whether or not the transaction adheres to the application team’s policy. If it does adhere, the transaction proceeds with the attestation attached, which allows the deposit to be executed. If the transaction doesn’t adhere to the policy, the address is unable to deposit funds to the protocol, thereby significantly reducing commingling risk for institutions using the protocol compliantly. Compliant users deposit funds unimpeded, and face no degradation of privacy guarantees.

## 6\) Open questions

- N/A

## 7\) Notes and links

- Relevant use cases where programmable policy can enhance compliance:  
  - [Private stablecoins](https://github.com/ethereum/iptf-map/blob/master/use-cases/private-stablecoins.md)  
  - [Private RWA Tokenization](https://github.com/ethereum/iptf-map/blob/master/patterns/pattern-private-vaults.md)  
  - [Private Bonds](https://github.com/ethereum/iptf-map/blob/master/use-cases/private-bonds.md)  
- Predicate programmable policy platform [docs](https://docs.predicate.io/v2/essentials/overview)
