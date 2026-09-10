---
title: "Vendor: TX-SHIELD"
status: ready
---

# TX-SHIELD – Institutional Privacy Infrastructure

TX-SHIELD develops regulated privacy-preserving infrastructure for institutional payments (TX-SHIELD), private AI collaboration (OpenTMP LLM), and MPC-TSS key management (Collab-Key). 
TX-SHIELD’s three core solutions, **TX-SHIELD**, **OpenTMP LLM**, and **Collab-Key**, address compliance, compute, and custody privacy challenges faced by regulated institutions.

---

## What it is
TX-SHIELD builds modular privacy layers/solutions for institutional finance and AI systems:
- **TX-SHIELD**: a regulated private payment layer/solution built on MPC-based encryption and threshold key control.  
It enables confidential settlement for stablecoins, RWAs, and bonds while ensuring regulator-auditable transparency.  
Transactions are visible only to stakeholders, while regulators can access details through granted audit keys.

- **OpenTMP LLM**: a distributed edge AI training and inference framework designed for privacy-preserving large-language model collaboration. It combines federated learning and multi-party computation (MPC-FL) to keep data local while enabling encrypted aggregation and joint model updates. It powers collaborative, effcient, secure, and governable AI training across distributed environments.

- **Collab-Key**: a high-performance MPC-TSS framework integrated with CrossBar’s EMPC + ReRAM hardware to provide a unified, privacy-preserving stack for institutional custody and payments. By executing multi-party ECDSA signing within a ReRAM-backed secure element, the solution ensures that full private keys are never reconstructed, eliminating single points of failure at both algorithmic and physical layers. Featuring an offline-by-default architecture and FIDO2 authentication, this integration provides high-assurance protection and regulator-auditable transparency for stablecoins, RWAs, and digital bonds.

Each module can operate independently or as part of a unified privacy-preserving stack across payments, compute, and custody.

---

## Fits with patterns

- [Private Stablecoin Shielded Payments](../patterns/pattern-private-stablecoin-shielded-payments.md)
- [Private PvP Stablecoins (ERC-7573)](../patterns/pattern-private-pvp-stablecoins-erc7573.md)
- [Regulatory Disclosure Keys & Proofs](../patterns/pattern-regulatory-disclosure-keys-proofs.md)
- [MPC Custody](../patterns/pattern-mpc-custody.md)

---

## Not a substitute for

TX-SHIELD:
- ZK-based L2 privacy frameworks (e.g., Aztec, Scroll)
- General-purpose MPC or TEE frameworks for secure computation (TX-SHIELD focuses on transactional privacy and compliance)
- Traditional on-chain settlement systems without regulator access


OpenTMP LLM:
- Centralized AI model training pipelines  
- Non-encrypted data-sharing frameworks  


Collab-Key:
- Single-key custodial wallets  
- Hardware-based key storage only 

---

## Architecture
### TX-SHIELD
Implements a high performance MPC-based private payment layer with threshold key control and an audit-key protocol for regulator visibility.  
Only sender, receiver i.e. stakers, and authorized regulators can access encrypted transaction details.  
Optimized MPC execution enables high performance (~10k TPS).

### OpenTMP LLM
Distributed AI architecture using federated learning and multi-party computation (MPC-FL) with threshold-secure aggregation.  
Supports edge acceleration, model distillation, quantization, and joint model governance.  

### Collab-Key
Supports 2PC and multi-party ECDSA protocols, integrated with CrossBar EMPC + ReRAM hardware.
Key shards are processed and stored within a ReRAM-backed secure element, ensuring private keys are never reconstructed in full at either the algorithmic or physical levels.
Features an offline-by-default architecture with native FIDO2 support, integrating seamlessly with institutional KMS and APIs.

---

## Privacy domains
- Private Payments / Compliance Infrastructure  
- Collaborative AI / Federated Learning Privacy  
- ReRAM-backed Institutional Custody / Key Management  

---

## Enterprise demand and use cases
TX-SHIELD:
- Institutional settlement for stablecoins, tokenized RWAs, and bonds on-chain.  
- Ideal for financial institutions needing confidentiality and compliance together.

OpenTMP LLM: 
- Privacy-preserving AI model training and inferences for enterprises and regulated sectors, such as healthcare, finance, and government. 

Collab-Key:
Institutional wallets, custodians, and signing infrastructure requiring fault-tolerant, hardware-secure (ReRAM) key management to eliminate single points of failure.

---

## Technical details
TX-SHIELD:
- MPC-based encryption, threshold key control (TSS), high-throughput multi-party computation.

OpenTMP LLM: 
- MPC-FL, Distributed Learning, Edge AI Acceleration, SFT, RLHF.  

Collab-Key:
- MPC-TSS (ECDSA 2PC / multi-party), Threshold Signatures, Secure Key Generation.   

---

## Strengths
- TX-SHIELD delivers institutional-grade transactional privacy with built-in regulatory visibility.  
- OpenTMP LLM enables privacy-preserving multi-party AI collaboration across edge and federated (hybrid on/off-chain) environments.  
- Collab-Key provides fault-tolerant, MPC-TSS-based signing with formal security foundations (*USENIX Security 2025*), strengthened by CrossBar ReRAM hardware-level isolation to eliminate physical-layer vulnerabilities.

---

## Risks and open questions
TX-SHIELD:
- Governance over regulator audit keys  
- Integration complexity across different blockchain environments  

OpenTMP LLM:
- Coordination complexity in multi-party settings  
- Trade-offs between model performance and full encryption overhead 

Collab-Key:
- Performance scaling with increased party count  
- Implementation complexity across heterogeneous custody systems
- Challenges in standardizing the interface between the ReRAM-backed secure element and diverse institutional legacy KMS/custody environments.
- Establishing protocols for secure shard migration or recovery in the event of physical hardware failure or decommissioning of the CrossBar EMPC module.  

---

## Links
Official Website: [TX-SHIELD](https://TX-SHIELD.com) | [BenPay|Privacy Wallet](https://www.benpay.com/wallet?type=privacy)

Contact: [ZYX Research](mailto:zyxresearch@gmail.com) | [haiyangxue@smu.edu.sg](mailto:haiyangxue@smu.edu.sg)

Papers:  
- [USENIX Security 2025 – Guo et al.](https://www.usenix.org/system/files/usenixsecurity25-guo-hao-improved.pdf)  
- [USENIX Security 2025 – Liang et al.](https://www.usenix.org/system/files/usenixsecurity25-liang-achilles.pdf)
