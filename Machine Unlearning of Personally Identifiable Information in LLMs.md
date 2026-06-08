# Machine Unlearning of Personally Identifiable Information in LLMs (D. Parii et al., 2025, ACL/NLLP)
# 大型語言模型中個人可識別資訊（PII）的機器遺忘技術：UnlearnPII 基準與 PERMU 方法的分析

### 核心問題與動機
大型語言模型（LLMs）在海量網路資料上預訓練，常會無意中記憶並重現**個人可識別資訊（Personally Identifiable Information, PII）**，如姓名、電話、地址、銀行帳號、醫療資訊等。這帶來嚴重的法律與倫理風險，尤其符合歐盟 GDPR 的「被遺忘權」（Right to be Forgotten），要求資料主體能請求刪除其個人資料。
傳統全量重新訓練成本極高且不具彈性，而現有機器遺忘（Machine Unlearning）方法多聚焦於一般事實或虛構作者資料（如 TOFU 基準），忽略 PII 的特性：
- **隱含知識（Implicit Knowledge）**：模型可能透過同義改寫、間接關聯或 jailbreak 攻擊洩露 PII。
- **評估不足**：現有指標常平等對待所有 token，忽略 PII 的細粒度洩露，且缺乏跨領域（如一般、銀行、醫療）與對抗性測試。
- **實用性挑戰**：方法需同時達成有效遺忘（低洩露率）、保留模型效用（Utility）、維持輸出品質，且易於整合到現有工作流程。
論文動機是開發**模型無關（model-agnostic）、計算高效**的 PII 專用遺忘解決方案，並建立專門基準來系統性評估，推動符合隱私法規的實際應用。研究問題包括：如何同時移除顯性與隱性知識？不同 PII 類別的遺忘難度差異？SOTA 方法在不同模型規模上的表現？
### 結果/成果
**1. UnlearnPII 基準**：
- 包含 225 個合成個人檔案（2000+ QA 對），涵蓋 16 種 PII 類別，跨一般、銀行、醫療三大領域。
- 評估框架：內範圍攻擊（DirectQA、ParaphrasedQA）與外範圍攻擊（OneHopQA、InvertedQA），透過自動補全（Autocompletion）與提取攻擊測試。
- 細粒度指標：Extraction Success Rate (ESR)，區分顯性/隱性洩露，同時測量 Utility、Fluency 與保留集（Test Retain）表現。
- 優點：解決現有基準忽略隱含知識與對抗穩健性的問題，提供更現實的 PII 風險評估。
**2. PERMU_tok 方法**（PERMU 的 token-level 擴展）：
- 基於原始 PERMU（Perturbation-based Machine Unlearning），透過 token-level 噪音注入生成對抗樣本，結合對比學習調整 logit 分布。
- 改進點：用簡單啟發式（以目標人物姓名作為 subject token）取代計算密集的 Model Sensitivity Metric (MSM)；轉為 token-level 噪音，實現模型無關（只需修改輸入資料，無需改動模型 forward 函數）。
- 參數：Replace Token Probability (R=1) 與 Corrupt Token Neighborhood (N=k1_match)，平衡遺忘強度與效用。
- **主要成果**（以 Llama3.1–8B 為例）：
 - Direct/Paraphrased 攻擊 ESR 降至 0.22%–0.61%（顯著優於基線）。
 - 隱性攻擊（如 InvertedQA）也有良好表現。
 - Test Retain ESR 維持 >95%，Utility 輕微下降但在 MMLU、GSM8K、ARC 等通用基準上僅掉 <1%。
 - 優於 Gradient Ascent (GA)、DPO、NPO 等替代方法（後者常導致災難性遺忘或輸出退化）。
- 跨模型規模測試（Qwen2.5 1.5B–32B）：更大模型傾向有更好遺忘效果與知識分離能力。
- 開源程式碼公開可用，易於整合。
不同 PII 類別表現有差異：電話號碼等孤立識別符較易遺忘；職業、疾病、治療等語義豐富類別較難完全移除（ESR 殘留 5–9%），因其形成廣泛關聯網路。
### 分析與洞見
- **遺忘 vs. 效用權衡**：PERMU_tok 透過溫和 token-level 擾動，產生更高熵的對抗分布，有效漂移概念而非死記硬背，適合 PII 這種需要移除「關聯」而非單一事實的場景。相較 embedding-level 原始 PERMU，它在隱性知識移除上更優，效用損失更小。
- **PII 語義特性**：語義豐富的 PII 形成多路徑記憶，更難精準切斷。這暗示未來需結合語義圖或更細粒度遺忘策略。
- **模型規模影響**：更大模型因參數容量大，更易分離目標知識與通用知識，符合 scaling law 直覺。但小模型在特定設定下也展現潛力。
- **評估細微之處**：精確匹配（exact matching）用於 ESR 避免模糊匹配的假陽性，但可能低估部分洩露。合成資料雖控制良好，但現實中 PII 稀疏，遺忘效果預期更好。
- **邊緣案例與限制**：
 - 未達「完全」遺忘，特別在對抗性 jailbreak 下仍有殘留風險。
 - 訓練設定（多 epoch 專注 PII 微調）放大遺忘挑戰，但不完全反映真實世界（PII 稀疏）。
 - 基準未涵蓋所有 GDPR 合規面向（如隱藏狀態分析、成員推斷攻擊）。
 - 其他方法（如 GA）易造成災難性遺忘或「我不知道」式退化，凸顯 PERMU 家族的實用優勢。
**更廣泛意涵**：此工作橋接技術與法規需求，為企業/研究者提供可操作工具，降低隱私風險同時維持 LLM 效能。開源性促進社群迭代，未來可擴展至多模態或即時遺忘。
### 結論
論文成功推進 PII 機器遺忘領域，提出 UnlearnPII 基準與實用 PERMU_tok 方法，證明可在保留模型效用的前提下大幅降低洩露風險，特別在顯性知識移除上表現優異。同時揭示語義豐富 PII 的挑戰與模型規模的潛在優勢，為 GDPR 等法規合規提供重要技術支柱。
雖然未達成絕對完美遺忘，但這是朝向可靠、模型無關解決方案的重要一步。未來方向包括更穩健的模糊評估、現實稀疏資料測試、跨領域擴展，以及探索 scaling law 與混合方法。整體而言，此研究為 LLM 隱私治理貢獻了可落地且具啟發性的框架
**論文連結**： 
- ACL Anthology 主頁：https://aclanthology.org/2025.nllp-1.6/ 
- PDF 下載：https://aclanthology.org/2025.nllp-1.6.pdf
