# R.R.: Unveiling LLM Training Privacy through Recollection and Ranking (ACL 2025)
# 透過 Recollection 與 Ranking 揭露 LLM 訓練資料隱私漏洞

## 🎯 核心問題與動機

大型語言模型（LLM）在訓練過程中會**隱式記憶（implicit memorization）**大量原始資料，即使未出現明顯過擬合，仍可能洩漏敏感資訊。其中，**個人可識別資訊（PII）** 是最主要的隱私風險來源，包括姓名、地址、電話、Email 等。

### 現有隱私攻擊分類
- **Membership Inference Attack (MIA)**：僅判斷某資料是否在訓練集中，無法還原具體內容。
- **Data Extraction**：盡可能提取訓練資料，但難以針對特定目標。
- **PII Reconstruction**（本論文重點）：在已「擦除（scrubbed）」的訓練資料中（PII 被替換為 `[MASK]`），還原被遮罩的真實 PII 實體。

**這是最實際的威脅**，因為許多 LLM 開發者會公開 scrubbed 資料集供使用者參考或微調。

### 現有方法的局限
- Prefix continuation（如 TAB 方法）僅依賴前文，忽略後文上下文，多重遮罩時需截斷。
- Perplexity scoring 或 MLM 填充需事先知道遮罩長度，實務上難以應用。
- 整體 top-1 準確率通常低於 10%。

**論文動機**：在黑盒（black-box）API 存取情境下，設計更有效率的 PII 重建攻擊，證明 scrubbed 資料仍極易被攻破，呼籲社群重視此隱私風險。攻擊假設攻擊者可取得 scrubbed 文字，並知道 victim LLM 的 pre-trained reference model（常見於開源模型微調）。

## 🚀 R.R. 方法與實驗成果

**R.R.（Recollect and Rank）** 為雙階段攻擊框架：

### 1. Recollection（候選產生階段）
- 將完整 masked 文字輸入 victim LLM，以 prompt 指示「重述該文字，並填入遮罩」。
- 充分利用前後文脈（bidirectional-like context）。
- 重複多次產生多樣輸出，使用 Microsoft Presidio（NER-based PII 識別器）提取候選，形成候選池。
- **優勢**：符合 LLM next-token prediction 訓練特性，查詢次數少即可達到高 recall。

### 2. Ranking（候選排序階段）
- 將候選插入 `[MASK]` 位置，計算 **partial cross-entropy loss**（僅計算 PII 開始的後續 tokens，降低成本）。
- 提出 **biased reference calibration** 新準則：  
  **C(M) = L(M) + b × (L(M) - Lr(M))**  
  （L：victim model loss，Lr：reference model loss，b 為可調偏置）
- 多重遮罩時採用 greedy 分段求和，大幅降低計算複雜度。
- 理論證明該準則可同時保留兩種基準優勢，並透過 b 彈性調整。

### 實驗成果
- **資料集**：ECHR、ENRON、LLM-PC
- **模型**：四種主流 LLM
- **LLM-PC 資料集** top-1 準確率平均 **25.73%**
- 相較先前 SOTA 提升 **超過 100%**（最高 +148%），整體平均提升 **122%**
- Recollection 階段查詢更有效率；Ranking 階段可忽略前文，進一步減少 API 呼叫
- 即使 reference model 不完全正確，效能下降也很小
- **結論**：scrubbed 資料仍高度脆弱

## 💡 分析與洞見

- **上下文利用的重要性**：充分利用雙向上下文，大幅優於僅用 prefix 的傳統方法，顯示 LLM 記憶具有全局關聯特性。
- **Loss 作為 ranking 信號**：Cross-entropy loss 直接反映模型對 PII 的熟悉程度；biased calibration 是關鍵創新，能有效過濾 pre-training 干擾。
- **理論貢獻**：證明新準則可保留 L 與 Cr 的正確預測，重疊部分不遺失。
- **實務意涵**：
  - API 成本更低，更具可行性。
  - 對開發者：單純 scrubbing 已不足夠，需結合差分隱私、強去識別化或拒答機制。
  - 潛在風險：攻擊者可針對公開 scrubbed 資料集大規模重建，威脅企業自訂模型。

### 局限與未來方向
- 準確率仍非 100%，對極稀有 PII 或強去識別化資料可能失效。
- 依賴 NER 識別器品質。
- 未來可延伸至多模態、其他 inference attack，或結合 MIA 技術。

## 📝 結論

論文提出 **R.R.** 框架，有效揭露 LLM 訓練隱私漏洞，證明即使經過 PII 遮罩的資料仍可被精準重建。這不僅是技術突破，更是對 LLM 隱私安全的重大警鐘：

> **記憶化是 LLM 的本質特性，單純 scrubbing 無法完全防護。**

論文同時釋出程式碼與資料集，期待社群共同發展更 robust 的隱私保護機制。對於研究者與開發者而言，此文是理解 LLM PII 洩漏風險與攻擊手法的重要參考。

---

**論文連結**： 
- arXiv: https://arxiv.org/abs/2502.12658 (或 PDF: https://arxiv.org/pdf/2502.12658) 
- ACL Anthology: https://aclanthology.org/2025.findings-acl.894/
