# Hidden No More: Attacking and Defending Private Third-Party LLM Inference

> **ICML 2025** | [Paper Link](https://proceedings.mlr.press/v267/thomas25b.html) | [Attack Paper (arXiv)](https://arxiv.org/abs/2505.18332) | [Cascade Paper (arXiv)](https://arxiv.org/abs/2507.05228)

---

## 🇬🇧 English Version

### 📖 Overview

This repository provides a summary and analysis of the ICML 2025 paper **"Hidden No More: Attacking and Defending Private Third-Party LLM Inference"** by Rahul Thomas, Louai Zahran, Erica Choi, Akilesh Potti, Micah Goldblum, and Arka Pal.

**The TL;DR:** Permutation-based "privacy" for LLM inference isn't private at all. The authors introduce a novel attack that reconstructs original prompts from hidden states with **near-perfect accuracy** — and then propose **Cascade**, an efficient defense that actually works.

---

### 🔥 Key Contributions

#### 1. The Attack: Prompt Reconstruction from Hidden States

The authors demonstrate that **causal attention** leaves positional fingerprints all over LLM hidden states. Their reconstruction technique achieves:

- **Near-perfect accuracy** across multiple state-of-the-art open-weight LLMs
- **Breaks three recently proposed privacy schemes** that rely on permutation and noise-based defenses
- **Exposes fundamental flaws** in prior theoretical "proofs" of permutation security

**Why this matters:** If you're using a third-party LLM service that claims to protect your prompts via permutation or obfuscation — your prompts can be recovered.

#### 2. The Defense: Cascade Protocol

Cascade is a **multi-party inference protocol** that leverages **sharding in the sequence dimension** to maintain privacy.

| Feature | Cascade | SMPC | Permutation-based |
|---------|---------|------|-------------------|
| **Privacy Guarantee** | Empirical | Provable | ❌ Broken |
| **Speed** | ⚡ Orders of magnitude faster | 🐢 1000× slower | Fast |
| **Scalability** | ✅ Scales to modern LLMs | ❌ Doesn't scale | Scales |
| **Resistant to Known Attacks** | ✅ Yes | ✅ Yes | ❌ No |

> Cascade trades off cryptographic privacy guarantees for **increased performance and scalability** — and it's **orders of magnitude faster than existing schemes** while remaining **secure against both this attack and previous methods**.

---

### 📊 Attack Visualization (Conceptual)

```
User Prompt: "What is the capital of France?"

        ↓ (sent to third-party with permutation-based "privacy")

Third-party receives: [permuted hidden states]

        ↓ (attacker applies reconstruction technique)

Attacker recovers: "What is the capital of France?"  ← Near-perfect reconstruction!
```

---

### 🛠️ Who Should Care

| Audience | Why It Matters |
|----------|----------------|
| **LLM Service Providers** | Your "privacy" claims may be false. Audit your defenses. |
| **AI/ML Researchers** | Statistical obfuscation ≠ security. Test your assumptions. |
| **Application Developers** | User prompts sent to third-party APIs may not be private. |
| **Security Engineers** | New attack surface: model hidden states leak input data. |

---

### 📚 Paper Details

- **Full Title:** Hidden No More: Attacking and Defending Private Third-Party LLM Inference
- **Conference:** ICML 2025 (Proceedings of Machine Learning Research, Vol. 267, Pages 59434–59469)
- **Authors:** Rahul Krishna Thomas, Louai Zahran, Erica Choi, Akilesh Potti, Micah Goldblum, Arka Pal
- **Attack Paper:** [arXiv:2505.18332](https://arxiv.org/abs/2505.18332) — *An Attack to Break Permutation-Based Private Third-Party Inference Schemes for LLMs*
- **Cascade Paper:** [arXiv:2507.05228](https://arxiv.org/abs/2507.05228) — *Cascade: Token-Sharded Private LLM Inference*
- **ICML Page:** [https://icml.cc/virtual/2025/poster/45330](https://icml.cc/virtual/2025/poster/45330)

---

### 📖 Citation

```bibtex
@InProceedings{pmlr-v267-thomas25b,
  title = {Hidden No More: Attacking and Defending Private Third-Party {LLM} Inference},
  author = {Thomas, Rahul Krishna and Zahran, Louai and Choi, Erica and Potti, Akilesh and Goldblum, Micah and Pal, Arka},
  booktitle = {Proceedings of the 42nd International Conference on Machine Learning},
  pages = {59434--59469},
  year = {2025},
  volume = {267},
  series = {Proceedings of Machine Learning Research},
  publisher = {PMLR}
}
```

---

### 📌 Key Takeaway

> **"If your defense isn't attack-tested, it's not defense."**

This paper is a wake-up call for the LLM privacy community: **permutation-based schemes are broken**, and we need rigorous, attack-driven security analysis — not theoretical assumptions. Cascade offers a promising path forward.

---

---

## 🇭🇰 繁體中文版本

### 📖 概述

本倉庫提供 ICML 2025 論文 **《Hidden No More: Attacking and Defending Private Third-Party LLM Inference》** 的摘要與分析，作者為 Rahul Thomas、Louai Zahran、Erica Choi、Akilesh Potti、Micah Goldblum 及 Arka Pal。

**一句話總結：** 基於置換（permutation）的 LLM 推理「隱私保護」一點也不隱私。作者提出了一種新穎的攻擊方法，能從隱藏狀態中以**近乎完美的精度**重建原始提示詞——隨後提出了**Cascade**，一套真正有效的防禦方案。

---

### 🔥 核心貢獻

#### 1. 攻擊：從隱藏狀態重建提示詞

作者證明 **因果注意力機制（causal attention）** 會在 LLM 的隱藏狀態中留下位置指紋。其重建技術達成：

- 在多個最先進的開源權重 LLM 上實現**近乎完美的準確率**
- **攻破三種近期提出的隱私方案**，這些方案依賴於置換與雜訊防禦
- 揭露先前置換安全性的理論「證明」中存在**根本性缺陷**

**為何重要：** 如果您使用的第三方 LLM 服務聲稱透過置換或混淆來保護您的提示詞——您的提示詞可以被還原。

#### 2. 防禦：Cascade 協議

Cascade 是一種**多方推理協議**，利用**序列維度的分片（sharding）** 來維護隱私。

| 特性 | Cascade | SMPC | 基於置換的方案 |
|------|---------|------|---------------|
| **隱私保證** | 實證層級 | 可證明 | ❌ 已被攻破 |
| **速度** | ⚡ 快數個數量級 | 🐢 慢 1000 倍 | 快速 |
| **可擴展性** | ✅ 可擴展至現代 LLM | ❌ 無法擴展 | 可擴展 |
| **抵抗已知攻擊** | ✅ 是 | ✅ 是 | ❌ 否 |

> Cascade 以犧牲加密級別的隱私保證為代價，換取**更高的效能與可擴展性**——它比現有方案**快數個數量級**，同時**能抵禦本文攻擊及先前所有已知攻擊**。

---

### 📊 攻擊概念圖

```
用戶提示詞：「法國的首都是什麼？」

        ↓（發送給第三方，經過「基於置換的隱私保護」）

第三方收到：[置換後的隱藏狀態]

        ↓（攻擊者應用重建技術）

攻擊者還原出：「法國的首都是什麼？」  ← 近乎完美的重建！
```

---

### 🛠️ 誰需要關注

| 對象 | 為何重要 |
|------|---------|
| **LLM 服務提供商** | 您的「隱私」聲明可能是虛假的。請稽核您的防禦機制。 |
| **AI/ML 研究者** | 統計混淆 ≠ 安全性。請檢驗您的假設。 |
| **應用開發者** | 發送至第三方 API 的用戶提示詞可能並非私有。 |
| **安全工程師** | 新的攻擊面：模型隱藏狀態會洩漏輸入資料。 |

---

### 📚 論文資訊

- **完整標題：** Hidden No More: Attacking and Defending Private Third-Party LLM Inference
- **會議：** ICML 2025（Proceedings of Machine Learning Research，第 267 卷，第 59434–59469 頁）
- **作者：** Rahul Krishna Thomas、Louai Zahran、Erica Choi、Akilesh Potti、Micah Goldblum、Arka Pal
- **攻擊論文：** [arXiv:2505.18332](https://arxiv.org/abs/2505.18332) — *An Attack to Break Permutation-Based Private Third-Party Inference Schemes for LLMs*
- **Cascade 論文：** [arXiv:2507.05228](https://arxiv.org/abs/2507.05228) — *Cascade: Token-Sharded Private LLM Inference*
- **ICML 頁面：** [https://icml.cc/virtual/2025/poster/45330](https://icml.cc/virtual/2025/poster/45330)

---

### 📖 引用格式

```bibtex
@InProceedings{pmlr-v267-thomas25b,
  title = {Hidden No More: Attacking and Defending Private Third-Party {LLM} Inference},
  author = {Thomas, Rahul Krishna and Zahran, Louai and Choi, Erica and Potti, Akilesh and Goldblum, Micah and Pal, Arka},
  booktitle = {Proceedings of the 42nd International Conference on Machine Learning},
  pages = {59434--59469},
  year = {2025},
  volume = {267},
  series = {Proceedings of Machine Learning Research},
  publisher = {PMLR}
}
```

---

### 📌 核心啟示

> **「如果您的防禦未經攻擊測試，那它就不算防禦。」**

這篇論文是對 LLM 隱私社群的一記警鐘：**基於置換的方案已經被攻破**，我們需要嚴格的、以攻擊驅動的安全分析——而非理論假設。Cascade 提供了一條有前景的前進之路。
