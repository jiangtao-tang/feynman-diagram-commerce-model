# Decentralized Commerce Feynman Diagram Model

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![GitHub Stars](https://img.shields.io/github/stars/jiangtao-tang/feynman-diagram-commerce-model?style=social)](https://github.com/jiangtao-tang/feynman-diagram-commerce-model)
[![SSRN Preprint](https://img.shields.io/badge/SSRN-Preprint-orange)](https://ssrn.com/abstract=XXXXXXX)

---

## 📖 Overview

This repository contains the complete **Whitepaper** of the *"Decentralized Commerce Feynman Diagram Model"*, available in both **Chinese** and **English**.

The model introduces the conceptual framework of **Quantum Field Theory's Feynman Diagrams** into decentralized commerce modeling, proposing a **programmable credit system** implemented through dNFT pairs (+dNFT / -dNFT) that realizes **double-entry bookkeeping on-chain**.

---

## 📂 Files

| File | Description |
|------|-------------|
| `Feynman_Diagram_Model_for_Decentralized_Commerce_Whitepaper_v1.2_EN.md` | Full English whitepaper (10 chapters + 4 appendices) |
| `去中心化商业费曼图模型_白皮书_完整版_v1.2.md` | Full Chinese whitepaper (完整版) |
| `Economic_Analysis_of_Decentralized_Commerce_Trust_Mechanism_EN.md` | English economics paper |
| `assets/` | Feynman diagram images and figures |

---

## 🔬 Core Innovations

1. **First application of the complete Feynman diagram conceptual framework** (vertices, propagators, conservation laws, creation/annihilation) to decentralized commerce modeling.

2. **dNFT pair mechanism** (+dNFT as asset / -dNFT as liability) that directly implements **double-entry bookkeeping** debit/credit mechanism on-chain **without third-party records**.

3. **Dual-world conservation laws** (Real layer ↔ Virtual layer), ensuring Σ(dNFT) = 0 and Σ(asset value) = const.

4. **Local interaction vertices** (transaction contract + receipt contract), resolving the "locality dilemma" of remote transactions.

5. **Quantitative trust cost analysis** demonstrating **90-98% cost reduction** compared to centralized platforms (5-25% commission → $0.50/transaction).

---

## 🎯 Key Contributions

### 1. Theoretical Framework
- Maps QFT concepts to commerce: vertices = transaction contracts, propagators = asset transfers, conservation laws = accounting net-value balance.
- Feynman rules translated into smart contract logic.

### 2. Trust Cost Quantification
- Remote transactions have **15-30% higher trust cost** than face-to-face transactions.
- Two-local-vertex model reduces per-transaction trust cost to **$0.50** vs. platform commissions of 5-25%.

### 3. Game-Theoretic Proof
- When discount factor × future value ≥ fraud gain, **honesty is the optimal strategy**.
- Empirical result: at 96% discount factor, honest strategy present value is **120×** that of fraud.

### 4. Programmable Credit System
- Three-dimensional initial credit generation: **asset staking (40%) + VC endorsement (30%) + social guarantee (30%)**.
- New merchants can obtain an initial credit score of **75/100**.
### 5. Compatible with ERC-8004 / ERC-8126 / ERC-8183
---

## 📊 Feynman Diagram Analogy

| QFT Concept | Symbol | Commerce Model Analogy | Symbol |
|-------------|--------|----------------------|--------|
| Fermion (electron) | e⁻, e⁺ | dNFT (± credit quantum) | +dNFT, -dNFT |
| Gauge boson (photon) | γ | Logistics propagator | G_logistics |
| Interaction vertex | Γ | Contract execution point (local) | V_trade, V_annihilate |
| Pair creation | 0 → e⁺e⁻ | Merchant creates dNFT pair | 0 → (+dNFT) + (-dNFT) |
| Pair annihilation | e⁺e⁻ → γγ | Receipt contract, dNFT pair cancel | (+dNFT) + (-dNFT) → 0 |
| Propagator | G(x→y) | Logistics/credit propagator | G_logistics, G_credit |
| Charge conservation | ΣQ = const | dNFT number conservation | Σ(dNFT) = 0 |
| Energy-momentum conservation | Σpᵅ = const | Asset value conservation | Σ(value) = const |
| Scattering amplitude | ℳ | Transaction completion probability | P_trade |
| Feynman rules | diagram → integral | Contract rules → code execution | Smart contract logic |
| Running coupling | α(Q²) | Dynamic credit score | Score(t) |
| Renormalization | divergence → finite | Dispute arbitration → corrected distribution | Kleros ruling |

---

## 🏗️ Technical Architecture (5-Layer System)

```
Layer 5: Application Interface (React + Wagmi + dNFT Lifecycle Demo)
Layer 4: Transaction Execution (Escrow Contract + State Machine)
Layer 3: Programmable Credit (dNFT Core + Credit Scoring Contract) ← ★ Core
Layer 2: Data Notarization (Arbitrum L2 + IPFS)
Layer 1: Identity & Key Management (DID + Social Recovery Wallet)
```

---

## 🔍 Novelty Statement

**This work is the first to transplant the complete conceptual system of Feynman diagrams into decentralized commerce modeling.**

| Comparison | Existing Work | This Paper |
|-----------|---------------|------------|
| Mastromatteo et al. (2024) | Feynman diagrams as **perturbative calculation tool** (quantum finance) | Feynman diagrams as **conceptual framework** (vertex = transaction, propagator = transfer) |
| Triple-Entry Accounting (2019/2024) | Blockchain as **third-party recording layer** | dNFT pair's **inherent duality** implements double-entry bookkeeping without third party |
| Existing decentralized e-commerce | No credit system (cold start problem) | **Programmable credit** with initial credit generation |

---

## 👥 Authors

- **Jiang Tao** (姜涛) — Original Feynman Diagram Commerce Model  
  *Southwest University of Science and Technology*  
  ORCID: [0009-0003-9749-6697](https://orcid.org/0009-0003-9749-6697)

- **Mocha** — Formalization & Implementation  
  *Independent Researcher*  
  Smart contract development & mathematical formalization

---

## 📄 License

This work is licensed under **CC BY-NC-SA 4.0** (Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International).

- ✅ **Free to share and adapt** (with attribution)
- ❌ **Not for commercial use**
- 🔄 **Derivatives must use the same license** (ShareAlike)

[![License: CC BY-NC-SA 4.0](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-nc-sa.eu.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

---

## 🔗 Related Links

- 📄 **SSRN Preprint**: [Link to be updated after submission]
- 📄 **ChinaXiv Preprint** (Chinese version): [Link to be updated]
- 💻 **dNFT Demo (Vite + React + Wagmi)**: [Live demo link to be updated]
- 📖 **Full Whitepaper (English)**: `Feynman_Diagram_Model_for_Decentralized_Commerce_Whitepaper_v1.1_EN.md`
- 📖 **Full Whitepaper (Chinese)**: `去中心化商业费曼图模型_白皮书_完整版_v1.1.md`

---

## 📚 Citation

If you use this work, please cite:

```bibtex
@techreport{JiangTao2026Feynman,
  author = {Jiang, Tao and Mocha},
  title  = {An Economic Analysis of Decentralized Commerce Trust Mechanism Based on Feynman Diagram Model},
  year   = {2026},
  month  = {June},
  note   = {SSRN Preprint},
  url    = {https://github.com/jiangtao-tang/feynman-diagram-commerce-model}
}
```

---

## 📮 Contact

- **Jiang Tao**: [GitHub Issues](https://github.com/jiangtao-tang/feynman-diagram-commerce-model/issues) or email via GitHub profile.

---

## ⭐ Star History

If you find this work valuable, please consider **starring** this repository!

[![Star History Chart](https://api.star-history.com/svg?repos=jiangtao-tang/feynman-diagram-commerce-model&type=Date)](https://star-history.com/#jiangtao-tang/feynman-diagram-commerce-model&Date)

---

## 📋 Table of Contents (English Whitepaper)

1. [Introduction](#chapter-1-introduction)
2. [Background and Related Work](#chapter-2-background--related-work)
3. [Core Theory — The Feynman Diagram Commerce Model](#chapter-3-core-theory)
4. [Technical Architecture — The Five-Layer System](#chapter-4-technical-architecture)
5. [RWA and dNFT Implementation](#chapter-5-rwa-and-dnft-implementation)
6. [Credit Scoring and Game-Theoretic Analysis](#chapter-6-credit-scoring--game-theoretic-analysis)
7. [Security Analysis](#chapter-7-security-analysis)
8. [Implementation Roadmap](#chapter-8-implementation-roadmap)
9. [Comparison with Existing Solutions](#chapter-9-comparison-with-existing-solutions)
10. [Conclusion and Future Outlook](#chapter-10-conclusion--future-outlook)
11. [Appendices](#appendices)
