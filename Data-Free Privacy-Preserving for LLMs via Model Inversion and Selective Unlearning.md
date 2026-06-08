# Data-Free Privacy-Preserving for LLMs via Model Inversion and Selective Unlearning
# 無資料選擇性遺忘：透過模型反演實現 LLM 的資料免隱私保護（DFSU）論文深度分析


### 核心問題與動機

大型語言模型（LLMs）在預訓練過程中會從海量網際網路資料中**無意記憶（memorize）敏感的個人可識別資訊（PII）**，如地址、醫療記錄、IP、裝置識別碼等。這導致嚴重的隱私風險：模型可能在推理時重現這些資訊，面臨提取攻擊（extraction attacks，如 prefix probing）、成員推斷（membership inference）等威脅，進而引發法律（例如「被遺忘權」）、倫理與部署安全問題。

傳統**機器遺忘（Machine Unlearning）**技術旨在從模型中移除特定資料的影響，但大多數方法（如 Gradient Ascent (GA)、Negative Preference Optimization (NPO)）高度依賴**原始訓練資料或明確的 forget set**。這在實際部署中往往不可行：

- 訓練資料可能是專有、受法律限制或規模太大而無法取得。
- 部署者通常僅持有模型權重，無法存取原始 corpus。
- 現有方法在資料不可得時無法應用，無法滿足後置（post-hoc）隱私修復需求。

**本文核心創新**：提出 **Data-Free Selective Unlearning (DFSU)** 框架，在**完全無原始訓練資料**的情況下，僅使用模型參數與防禦者對 PII 類型的先驗知識（e.g., IP 地址類型），實現針對性移除 PII，同時盡量保留模型通用能力。這解決了「資料依賴」瓶頸，將模型反演（traditionally 視為攻擊）轉化為防禦工具，體現了「以攻為守」的巧妙思路。

**動機的更深層背景**：LLMs 如同「機率資料庫」，模型容量越大，長尾敏感內容記憶越強。訓練時防護（如 DP-SGD）雖有保證，但無法 retroactive 修復已部署模型，且常犧牲效能。DFSU 提供了一條實務可行的後置修復路徑，尤其適用於開源或商用模型的隱私合規場景。

---

#### 結果／成果

DFSU 採用**三階段管線**（見論文 Figure 2）：

1. **反演模型訓練（Inversion Model Training）**：訓練一個 logit-based inverter（基於序列到序列 Transformer，如 Flan-T5），從目標 LLM 最終 token 的 log-probability distribution 重建輸入文本。實現高品質 pseudo-data 生成（F1 ~30%、BLEU ~15%）。

2. **偽 PII 合成與標註（Pseudo-PII Synthesis and Annotation）**：使用 entity-swapped candidates 查詢目標模型，提取 logits，再由 inverter 生成 pseudo-PII，並透過 few-shot prompting 自動標註 token-level privacy masks（敏感實體位置）。

3. **選擇性遺忘（Privacy-Selective Contrastive Unlearning, PSCU）**：在 LoRA（低秩適應）子空間中優化，凍結預訓練權重。引入**對比遮罩損失**（contrastive mask loss）：對敏感 token 最大化損失（遺忘），對上下文 token 最小化損失（保留效用）。這實現 token-level 精準控制，避免全局破壞。

**實驗設定**：

- **模型**：Pythia 系列（160M、410M、1.4B）。
- **資料**：AI4Privacy PII-Masking 資料集注入 WikiText-103（生成任務）與 MNLI（分類/推理任務）。
- **評估**：隱私指標（ERR、FRS、S-Exp、E-Hit，越低越好）；效用指標（PPL for generative, Accuracy for MNLI）。
- **基準**：Oracle（有原始資料的 PSCU）作為上限比較。

**主要成果**（Injection-Based Simulation）：

- DFSU 在所有規模上將 **ERR 降至 0.00%**，匹配或接近 Oracle。
- FRS、S-Exp、E-Hit 等指標接近 Oracle，證明 pseudo-data 足夠有效。
- 效用損失極小：WikiText PPL 僅微增（e.g., Pythia-410M 從 8.69 到 8.83）；MNLI Accuracy 與 Oracle 非常接近（e.g., 1.4B 模型 77.05% vs 77.21%）。
- **In-the-Wild 評估**：直接應用於未注入的生產 checkpoint，仍能有效降低 PII 相關提示的洩漏。

**消融與穩健性**：PSCU 優於傳統 GA；LoRA rank 等參數影響效用保留；遺忘訊號飽和快，少量 pseudo-data 即可達成顯著效果。整體實現了優異的**隱私-效用權衡**。

---

#### 分析與洞見

**技術優勢與創新點**：

- **資料免（Data-Free）** 是最大亮點，解決了現實部署痛點。將 inversion 從攻擊轉為防禦，是典範轉移。
- **Token-level Selective + Contrastive Loss + LoRA** 組合確保局部化干預，避免 catastrophic forgetting 或全局效能崩潰。LoRA 限制更新空間，提高效率與穩定性。
- **Pseudo-data 作為 surrogate**：雖然有 fidelity 損失，但實驗顯示足以驅動有效遺忘，證明模型內部表示已包含足夠 PII 模式資訊。
- **專案實作價值**：管線模組化（inverter 可跨規模重用），易於整合到現有 LLM 部署流程。適合 GitHub 專案：可實作 DFSU pipeline、提供 LoRA 微調腳本、pseudo-data 生成工具，並支援不同 PII 類型。

**限制與邊緣案例**：

- Inversion 品質依賴目標模型架構與 PII 類型；對極長尾或高度混淆的 PII，可能 surrogate 保真度不足。
- 計算成本：雖然 LoRA 高效，但 inverter 訓練與多階段流程仍需資源（相對於 inference 較重）。
- 對抗性：若攻擊者知曉 DFSU，可能設計 bypass；未完全解決「遺忘不徹底」或新攻擊向量。
- 泛化：主要在 Pythia 驗證，需更多模型（Llama 等）與真實世界多樣 PII 測試。
- 倫理/法律：合成 pseudo-PII 雖避免直接使用真實資料，但仍需確保不引入新偏誤或洩漏風險。

**更廣洞見**：

- 反映 LLM 記憶的本質：模型是壓縮的訓練分布，inversion 可「解壓」有用 surrogate。
- 對隱私法規（如 GDPR）有實務意義，提供部署後合規工具。
- 未來方向：結合其他編輯技術（如 model editing）、提升 inversion 保真度、探索 multi-modal 或更大型模型、自動化 PII 類型偵測。
- 專案延伸：可開發開源工具包，包含評估套件（ERR 等 metrics）、不同 LoRA 配置 benchmark，以及與 DP、聯邦學習的混合方案。邊緣案例如低資源裝置部署或即時 unlearning 值得探索。

---

#### 結論

提出 DFSU 框架，成功填補了資料不可得情境下的 LLM 隱私保護空白，透過模型反演合成 surrogate 並結合精準 token-level 選擇性遺忘，實現了與 Oracle 高度競爭的隱私-效用平衡。這不僅是技術貢獻，更是對後置隱私修復實務路徑的探索，為 LLM 部署中的合規與安全提供了可操作解決方案。

**文章連結**：

- arXiv: https://arxiv.org/abs/2601.15595  
- PDF: https://arxiv.org/pdf/2601.15595

