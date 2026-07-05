
# 🔒 SecureGate: Token-Level Gating for Safe PII Revelation in Federated LLMs

[![arXiv](https://img.shields.io/badge/arXiv-2602.13529-b31b1b.svg)](https://arxiv.org/abs/2602.13529)
[![ACL](https://img.shields.io/badge/ACL-2026-blue)](https://acl2026.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Made with](https://img.shields.io/badge/Made%20with-LoRA-red)](https://arxiv.org/abs/2106.09685)

> **Official implementation & paper summary for "SecureGate: Learning When to Reveal PII Safely via Token-Gated Dual-Adapters for Federated LLMs"**  
> *M. Shaaban et al., ACL 2026*

---

## 📖 Overview

Federated Learning (FL) enables collaborative Large Language Model (LLM) training across organizations, but LLM memorization inevitably leads to Personal Identifiable Information (PII) leakage. Moreover, there is a fundamental conflict between global generalization and local utility.

**SecureGate** introduces a privacy-aware federated fine-tuning framework that tackles this challenge via a **dual-adapter LoRA architecture** coupled with a **token-level gating module**. It achieves fine-grained privacy control *during inference*—dramatically reducing PII leakage without sacrificing downstream utility.

---

## ✨ Key Features

- **🛡️ Dual-Adapter Architecture**: Separates *safe* (global) knowledge from *revealing* (local, sensitive) knowledge.
- **🎯 Token-Level Gating**: Dynamically routes each token to the appropriate adapter based on context.
- **🔐 Privacy-Utility Trade-off Broken**: Achieves significant privacy gains while *improving* task performance.
- **⚡ Efficient & Lightweight**: Minimal computational and communication overhead for real-world FL systems.
- **🏥 Regulatory Compliance**: Ready for GDPR, HIPAA, and other data privacy regulations.

---

## 🏗️ Methodology

SecureGate extends Low-Rank Adaptation (LoRA) with a dual-path design:

### 1. Architecture Overview

```text
                    ┌─────────────────────────────────────┐
                    │          Input Tokens              │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │   Token-Controlled Gating Module    │
                    │  (Dynamic Router - Inference Only)  │
                    └──────────────┬──────────┬───────────┘
                                   │          │
                    ┌──────────────▼──┐  ┌────▼───────────┐
                    │   🔒 Secure    │  │   🔓 Revealing  │
                    │   Adapter      │  │   Adapter       │
                    │ (Global/Shared)│  │ (Local/Private) │
                    └──────────────┬──┘  └────┬───────────┘
                                   │          │
                    ┌──────────────▼──────────▼───────────┐
                    │        Pre-trained LLM              │
                    └─────────────────────────────────────┘
```

### 2. Training Phase (Federated Setting)

| Component | Description |
| :--- | :--- |
| **Secure Adapter** | Trained locally and **aggregated globally** across clients. Learns "cleaned," shareable knowledge. |
| **Revealing Adapter** | Trained locally but **NEVER aggregated**. Retains organization-specific, sensitive knowledge. |
| **FL Aggregation** | Only the Secure Adapter parameters are uploaded to the central server for averaging. |

### 3. Inference Phase

- The **Gating Module** processes the input sequence token-by-token.
- It determines whether the context requires sensitive PII or not.
- **Route A (Safe)**: Tokens use the Secure Adapter → General, safe output.
- **Route B (Revealing)**: Tokens use the Revealing Adapter → Authorized access to sensitive details.

---

## 📊 Key Results

Experiments conducted on real-world PII-annotated datasets across multiple LLM backbones demonstrate the superiority of SecureGate:

| Metric | Performance Gain |
| :--- | :--- |
| **Inference Attack Accuracy** | **↓ 31.66×** (reduction) |
| **Extraction Attack Recall** | **↓ 17.07×** (reduction) |
| **Routing Reliability** | **100%** correct adapter activation |
| **Computational Overhead** | **Minimal** (negligible gating cost) |
| **Utility Impact** | **Positive** (outperforms baseline tasks) |

> 🚀 **Key Insight**: SecureGate achieves a *win-win* scenario—privacy is enhanced **and** model utility is maintained or even improved.

---

## 💡 Why This Matters

1. **Paradigm Shift**: Moves from "sanitize first, use later" to "learn all, reveal on demand." Sensitive knowledge is preserved but access-controlled, much like a secure vault.
2. **Knowledge Isolation in FL**: Provides a blueprint for isolating shareable vs. private knowledge in heterogeneous federated settings.
3. **Fine-Grained Control**: Token-level granularity allows for nuanced handling of mixed contexts (e.g., one sentence contains both public and private tokens).

---

## 🚀 Getting Started (Conceptual)

While the official code is pending full release, the framework can be integrated as follows:

```python
# Pseudo-code for SecureGate inference
from securegate import SecureGateLLM

model = SecureGateLLM.from_pretrained("base-llama")
model.load_adapters(secure_path="secure.pt", revealing_path="revealing.pt")

# Inference with auto-routing
output = model.generate(
    "Patient John Doe (ID: 12345) was diagnosed with...",
    enable_gating=True
)
# Safe tokens use secure adapter; PII tokens trigger revealing adapter
```

### Recommended Hyperparameters

- **LoRA Rank (r)**: Start with `r=8` and tune based on privacy-utility trade-off.
- **Gating Threshold**: Calibrate on a validation set with known PII labels.
- **Aggregation Frequency**: Adjust based on data heterogeneity across clients.

---

## 📝 Citation

If you find this work useful for your research, please cite the original paper:

```bibtex
@inproceedings{shaaban2026securegate,
  title={SecureGate: Learning When to Reveal PII Safely via Token-Gated Dual-Adapters for Federated LLMs},
  author={Shaaban, M. and Elmahallawy, M.},
  booktitle={Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (ACL)},
  year={2026},
  archivePrefix={arXiv},
  primaryClass={cs.CR},
  eprint={2602.13529}
}
```

---

## 📫 Resources

- 📄 **Paper Link**: [arXiv:2602.13529](https://arxiv.org/abs/2602.13529)

