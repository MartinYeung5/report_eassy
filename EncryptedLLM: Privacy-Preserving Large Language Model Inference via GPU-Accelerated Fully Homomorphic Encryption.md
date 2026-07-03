# EncryptedLLM: Privacy-Preserving Large Language Model Inference via GPU-Accelerated Fully Homomorphic Encryption

---

# English Version

---

## 📖 Overview

As large language models (LLMs) become increasingly powerful, the computation required to run them is frequently outsourced to third-party cloud providers. While this saves clients' computational resources, it risks leaking sensitive user queries to the cloud provider. EncryptedLLM addresses this fundamental privacy challenge by implementing a **GPU-accelerated Fully Homomorphic Encryption (FHE)** scheme that executes GPT-2 forward passes entirely on encrypted data, achieving **over 200× speedup** compared to CPU baselines.

---

## 🔍 Core Research

### Problem Definition

The "inference-as-a-service" paradigm for LLMs creates a critical privacy dilemma: users must send their queries to cloud service providers, but sensitive queries—such as those related to healthcare or finance—pose significant security risks if transmitted in plaintext. While FHE offers a natural cryptographic solution—encrypt the query and evaluate the LLM homomorphically on the cloud machine—the computational overhead of FHE combined with the already expensive inference of LLMs has made encrypted LLM inference previously impractical.

### Innovation

1. **First GPU-Accelerated CKKS Implementation**: Built upon OpenFHE, this work presents the first GPU-accelerated implementation of the CKKS homomorphic encryption scheme.
2. **Homomorphic-Friendly Activation Function Approximations**: Novel polynomial and piecewise approximations for LLM non-linear activation functions (Sign, GeLU) that enable efficient computation within FHE's arithmetic constraints.
3. **End-to-End Encrypted GPT-2 Inference**: Complete forward pass of GPT-2 executed entirely in the encrypted domain, achieving full end-to-end privacy protection.

### Key Results

- Encrypted GPT-2 inference achieves **over 200× speedup** compared to CPU baselines.
- Inference time reduced from **~3 hours to ~1 minute**, bringing encrypted LLM inference into a practically usable time window.
- Maintains model output quality through carefully designed activation function approximations, achieving the critical balance between privacy, performance, and accuracy.

### Practical Significance

- **Finance & Healthcare**: Banks and medical institutions can securely outsource sensitive tasks (credit assessment, transaction analysis, patient data processing) to cloud environments.
- **Enterprise Data Protection**: Model owners protect proprietary weights while users protect query privacy.
- **Democratizing Privacy-Preserving AI**: Demonstrates the feasibility of large-scale encrypted inference within reasonable timeframes, paving the way for broader adoption of privacy-preserving AI.

---

## 🛠️ Technical Details

### Methodology Overview

EncryptedLLM's technical approach comprises three layers:

**Layer 1: Homomorphic Encryption Infrastructure.** Based on OpenFHE, the team implemented a GPU-accelerated version of the CKKS scheme—an FHE variant supporting approximate arithmetic on encrypted real/complex numbers, naturally suited for neural network forward computation.

**Layer 2: Homomorphic Activation Function Transformation.** Non-linear activation functions (Sign, GeLU) cannot be directly represented using FHE's addition and multiplication primitives. The approach employs:

- **Sign Function Approximation**: Construct composite polynomials of the form $h(x) = f_n^{(d_f)} \circ g_m^{(d_g)}(x)$, where $f_n$ and $g_m$ are polynomials of degrees $2n+1$ and $2m+1$, respectively.
- **GeLU Function Approximation**: Piecewise polynomial strategies using different polynomial orders across intervals.
- **Core Trade-off**: Lower polynomial degrees yield faster computation but greater accuracy loss.

**Layer 3: Encrypted GPT-2 Inference.** The GPU-accelerated FHE scheme, combined with homomorphic-friendly activation functions, executes the complete GPT-2 forward pass in the encrypted domain.

### Experimental Setup

| Dimension | Details |
|---|---|
| **Base Library** | OpenFHE open-source fully homomorphic encryption library |
| **Encryption Scheme** | CKKS (approximate arithmetic FHE) |
| **Target Model** | GPT-2 |
| **Hardware Acceleration** | GPU |
| **Baseline** | CPU-based FHE implementation |
| **Evaluation Metrics** | Inference latency, model accuracy (pre- vs. post-activation approximation) |

---

## 📊 Key Findings

1. **GPU Acceleration Is the Critical Enabler**: CPU-based FHE implementations required hours for encrypted LLM inference; GPU acceleration compresses this to minutes (~3 hours → ~1 minute)—a fundamental leap from "theoretically possible" to "practically viable."

2. **Activation Function Approximation Is the Core Trade-off**: The paper's primary technical contribution is not a novel GPU algorithm but a systematic exploration of how to efficiently approximate LLM non-linear activation functions within FHE's constraints. This approximation strategy directly determines the feasibility and output quality of encrypted inference.

3. **End-to-End Privacy Protection Is Now Achievable**: Throughout the entire cloud processing pipeline, the server learns nothing about the user's input, intermediate computations, or final output—all remain encrypted and decryptable only by the key holder.

---

## 💡 Deep Insights

### Insight 1: FHE + GPU Is the Inevitable Path Forward

With LLMs containing billions to trillions of parameters, even plaintext inference demands substantial compute. Under encryption, every operation is amplified by orders of magnitude. EncryptedLLM's key insight: **algorithmic optimization alone cannot bridge this gap**—hardware-level breakthroughs (GPU parallel computing) are essential. This aligns with industry trends toward privacy-preserving hardware acceleration.

### Insight 2: Activation Function Approximation—An Underappreciated Core Challenge

Homomorphic encryption is inherently "arithmetic encryption"—it natively supports addition and multiplication but not comparison, branching, or exponentiation. Modern neural networks, however, rely heavily on non-linear activation functions (ReLU, GeLU, Sigmoid, etc.). Designing efficient approximations of these functions within FHE's "arithmetic cage" is the field's central challenge. EncryptedLLM's methodology is broadly generalizable to other LLM architectures and deep neural networks.

### Insight 3: Breaking the "Impossible Triangle"

Long-standing tensions exist between privacy protection, computational efficiency, and model accuracy. EncryptedLLM addresses efficiency through GPU acceleration and balances accuracy through polynomial approximations, **preliminarily breaking this "impossible triangle."** While currently validated on GPT-2 (a relatively small model), the approach's scalability is promising—as GPU compute continues to grow and FHE schemes continue to improve, encrypted inference for larger models is within reach.

### Insight 4: Strong Industrial Momentum

Authors from **J.P. Morgan Chase** demonstrate that major financial institutions have urgent real-world needs for privacy-preserving AI. The financial sector's stringent data privacy and compliance requirements make EncryptedLLM a critical enabler for cloud-based financial AI.

---

## 🎯 Practical Applications

### For Researchers

1. **Monitor FHE Hardware Acceleration Trends**: GPU acceleration is just the beginning—FPGA, ASIC, and other specialized hardware will further reduce encrypted inference overhead.
2. **Activation Function Approximation Is a Core Optimization Target**: Designing better homomorphic-friendly activation functions for different model architectures is a key research direction.
3. **Start Small, Scale Up**: GPT-2 validation provides a methodological foundation for larger models (Llama, GPT-3, etc.)—extend systematically from this baseline.

### For Industry Practitioners

1. **High-Privacy Sectors Should Prioritize Adoption**: Finance, healthcare, and other privacy-sensitive domains have genuine needs and clients willing to pay for privacy protection.
2. **Leverage OpenFHE and the Open-Source Ecosystem**: EncryptedLLM's OpenFHE foundation demonstrates that open-source FHE libraries have reached practical industrial usability.
3. **Customize Performance-Accuracy Trade-offs**: Different applications have different latency and accuracy requirements—select activation function approximation strategies accordingly.

### For Policymakers

EncryptedLLM demonstrates that **cloud-based AI can be deployed without sacrificing privacy**. This provides technical feasibility evidence for data cross-border flows, medical data sharing, financial data outsourcing, and other sensitive scenarios—potentially enabling regulations to find better balance between "privacy protection" and "AI development."

---

## 📚 References

- **Original Paper**: [EncryptedLLM: Privacy-Preserving Large Language Model Inference via GPU-Accelerated Fully Homomorphic Encryption](https://proceedings.mlr.press/v267/de-castro25a.html), *Proceedings of the 42nd International Conference on Machine Learning (ICML 2025)*
- **Authors**: Leo de Castro, Daniel Escudero, Adya Agrawal, Antigoni Polychroniadou, Manuela Veloso
- **Conference Page**: [ICML 2025 Poster](https://icml.cc/virtual/2025/poster/45395)
- **ACM Digital Library**: [DOI: 10.5555/3780338.3780825](https://dl.acm.org/doi/10.5555/3780338.3780825)
- **Code**: Official code repository not yet publicly available (follow OpenFHE community updates)

---

# 繁體中文版本

---

## 📖 概述

隨著大語言模型（LLM）能力不斷增強，模型推理所需的大量計算日益被外包至第三方雲端。雖然這節省了客戶端的計算資源，但同時也增加了用戶敏感查詢洩漏給雲端服務商的風險。EncryptedLLM 透過實現**GPU 加速的全同態加密（FHE）** 方案來解決此一根本性隱私挑戰，在完全加密的數據上執行 GPT-2 前向傳播，相較於 CPU 基準實現了 **超過 200 倍** 的加速。

---

## 🔍 核心研究

### 問題定義

LLM 的「推理即服務」（inference-as-a-service）模式面臨根本性的隱私困境：用戶需將查詢發送至雲端服務商，但敏感查詢（如醫療、金融相關話題）以明文傳輸存在嚴重安全隱患。全同態加密雖能從根本上解決此問題——用戶加密查詢後上傳，雲端在同態下完成全部計算——但 FHE 的計算開銷與 LLM 本身高昂的推理成本疊加，使得密態 LLM 推理此前被認為不切實際。

### 創新方法

1. **首個 GPU 加速的 CKKS 實現**：基於 OpenFHE 庫，首次實現了 CKKS 同態加密方案的 GPU 加速。
2. **同態友好型激活函數近似**：針對 LLM 中的非線性激活函數（Sign、GeLU）設計多項式近似與分段近似，使其可在 FHE 的算術約束下高效計算。
3. **端到端加密 GPT-2 推理**：在完全密態下完成 GPT-2 模型的完整前向傳播，實現全鏈路隱私保護。

### 關鍵結果

- 密態 GPT-2 推理速度相比 CPU 基準 **提升超過 200 倍**
- 推理時間從 **約 3 小時縮短至約 1 分鐘**，使密態 LLM 推理首次進入實際可用的時間窗口
- 透過激活函數的近似策略，在顯著提升性能的同時 **保持了模型輸出質量**，實現了隱私保護與實用性的關鍵平衡

### 實際意義

- **金融與醫療場景**：銀行和醫療機構可安全地將客戶信用評估、交易分析、病患數據處理等敏感任務外包至雲端
- **企業數據保護**：模型擁有者可保護其專有模型權重，同時用戶可保護查詢隱私
- **推動隱私計算民主化**：證明了在合理時間內完成大規模密態推理的可行性，為更廣泛場景下的隱私保護 AI 鋪平道路

---

## 🛠️ 技術細節

### 方法概述

EncryptedLLM 的技術路線可分為三個層次：

**第一層：同態加密基礎設施。** 研究團隊基於 OpenFHE 開源庫，實現了 CKKS 方案（一種支持近似算術的 FHE 方案）的 GPU 加速。CKKS 方案支持對加密的實數／複數進行加法和乘法運算，天然適配神經網絡的前向計算。

**第二層：激活函數的同態化改造。** 神經網絡的非線性激活函數（如 Sign、GeLU）無法直接用 FHE 的加法和乘法表示，必須進行多項式近似或分段近似：

- **Sign 函數的近似**：構造複合多項式 $h(x) = f_n^{(d_f)} \circ g_m^{(d_g)}(x)$，其中 $f_n$ 和 $g_m$ 分別為 $2n+1$ 階和 $2m+1$ 階多項式
- **GeLU 函數的近似**：採用分段多項式策略，在不同區間使用不同階數的多項式進行擬合
- **核心權衡**：多項式階數越低，計算越快，但精度損失越大

**第三層：加密 GPT-2 推理。** 將上述 GPU 加速的 FHE 方案與同態友好的激活函數結合，在密態下完成 GPT-2 的全部前向計算。

### 研究設定

| 設定維度 | 具體內容 |
|---|---|
| **基礎庫** | OpenFHE 開源全同態加密庫 |
| **加密方案** | CKKS（支持近似算術的 FHE 方案） |
| **目標模型** | GPT-2 |
| **硬體加速** | GPU |
| **基線對比** | CPU 上的 FHE 實現 |
| **評估指標** | 推理延遲、模型精度（激活函數近似前後的對比） |

---

## 📊 主要發現

1. **GPU 加速是密態 LLM 推理的關鍵突破點**：此前 FHE 的 CPU 實現在密態下執行 LLM 推理需要數小時，而 GPU 加速將其壓縮至分鐘級（從約 3 小時降至約 1 分鐘），這是從「理論可行」到「實際可用」的本質跨越。

2. **激活函數近似是性能與精度的核心權衡**：論文的核心技術貢獻並非某種全新的 GPU 算法，而是系統性地探索了如何在 FHE 的約束下高效近似 LLM 的非線性激活函數。這種近似策略直接影響密態推理的可行性和輸出質量。

3. **端到端隱私保護成為現實**：用戶的加密查詢在雲端處理的全過程中，雲端服務器無法獲知任何明文信息——既看不到用戶的輸入，也看不到中間計算結果，最終返回的仍是密文結果，僅持有私鑰的用戶才能解密。

---

## 💡 深度洞察

### 洞察一：FHE + GPU 的組合是隱私保護 AI 的必經之路

LLM 的參數量級（數十億至數萬億）意味著即使在明文下，推理也需要大量算力。在密態下，每個操作的开銷被放大數個數量級。EncryptedLLM 的關鍵 insight 在於：**僅僅依靠算法優化無法彌合這一鴻溝，必須從硬體層面（GPU 並行計算）尋求突破**。這與 NVIDIA 等廠商正在推動的「隱私計算硬體加速」趨勢高度一致。

### 洞察二：激活函數近似——被低估的核心技術挑戰

同態加密本質上是「算術加密」——它天然支持加法和乘法，但不支持比較、分支、指數等操作。而現代神經網絡依賴大量非線性激活函數（ReLU、GeLU、Sigmoid 等）。如何在 FHE 的「算術牢籠」中高效逼近這些非線性函數，是整個領域的核心難題。EncryptedLLM 在這個方向上的探索具有普適性——其方法論可推廣至其他架構的 LLM 乃至更廣泛的深度神經網絡。

### 洞察三：隱私保護與模型性能的「不可能三角」正在被打破

長期以來，隱私保護、計算效率和模型精度三者之間存在難以調和的張力。EncryptedLLM 透過 GPU 加速解決了效率問題，透過多項式近似平衡了精度問題，**初步打破了這一「不可能三角」**。雖然目前僅驗證了 GPT-2（相對較小的模型），但技術路線的可擴展性值得關注——隨著 GPU 算力的持續提升和 FHE 方案的不斷優化，更大規模模型的密態推理並非遙不可及。

### 洞察四：工業界的強力推動

該論文的作者來自 **摩根大通（J.P. Morgan Chase）** ，這表明大型金融機構對隱私保護 AI 有強烈的現實需求。金融行業對數據隱私和合規性的要求極為嚴格，EncryptedLLM 的出現為金融 AI 的雲端化掃除了關鍵障礙。

---

## 🎯 實踐應用

### 對研究者的建議

1. **關注 FHE 硬體加速趨勢**：GPU 加速只是起點，未來 FPGA、ASIC 等專用硬體的加入將進一步降低密態推理的開銷
2. **激活函數近似是核心優化方向**：針對不同模型架構設計更優的同態友好型激活函數，將是隱私保護 AI 領域的重要研究方向
3. **從小模型起步**：GPT-2 的驗證為更大模型（如 Llama、GPT-3 等）的密態推理提供了方法論基礎，可在此基礎上逐步擴展

### 對工業界的建議

1. **金融、醫療等高隱私行業應優先布局**：這些領域對數據隱私有剛需，且客戶願意為隱私保護支付溢價
2. **關注 OpenFHE 等開源生態**：EncryptedLLM 基於 OpenFHE 實現，表明開源 FHE 庫已具備一定的工業可用性
3. **性能與精度的權衡需按場景定制**：不同應用場景對延遲和精度的要求不同，可根據實際需求選擇合適的激活函數近似策略

### 對政策制定者的啟示

EncryptedLLM 展示了 **技術可以在不犧牲隱私的前提下實現 AI 的雲端化**。這為數據跨境流動、醫療數據共享、金融數據外包等敏感場景提供了技術可行性依據，有望推動相關法規在「隱私保護」與「AI 發展」之間找到更優的平衡點。

---

## 📚 參考資料

- **原始論文**: [EncryptedLLM: Privacy-Preserving Large Language Model Inference via GPU-Accelerated Fully Homomorphic Encryption](https://proceedings.mlr.press/v267/de-castro25a.html), *Proceedings of the 42nd International Conference on Machine Learning (ICML 2025)*
- **作者**: Leo de Castro, Daniel Escudero, Adya Agrawal, Antigoni Polychroniadou, Manuela Veloso
- **會議主頁**: [ICML 2025 Poster](https://icml.cc/virtual/2025/poster/45395)
- **ACM Digital Library**: [DOI: 10.5555/3780338.3780825](https://dl.acm.org/doi/10.5555/3780338.3780825)
- **相關程式碼**: 截至目前，官方程式碼庫尚未公開（建議關注 OpenFHE 社區後續更新）
