# Effective PII Extraction from LLMs through Augmented Few-shot Learning (S. Cheng et al., USENIX Security 2025)
# 透過增強式 Few-Shot Learning 實現高效 PII 從大型語言模型中提取

### 1. 核心問題與動機

大型語言模型（LLMs）在訓練時大量攝取網路資料，其中包含大量**個人識別資訊（PII）**，如姓名、電子郵件、電話號碼、職業等。這些 PII 可能來自公開來源、資料外洩或未經同意的收集，導致模型「記住」並能在提示下重現敏感資料，帶來嚴重隱私風險（例如 spear-phishing、騷擾或身分盜用）。

#### **現有方法的局限**

- **Jailbreak**：輸出不穩定，常產生虛構 PII，且易被對齊機制阻擋。
- **Fine-tuning**：依賴模型提供 fine-tuning 接口，一旦不可用即失效。
- **Direct Querying**：成功率低，尤其在非目標（non-targeted）提取大量 PII 時效率不佳；針對性（targeted）提取也受限於部分已知資訊。
- 非目標提取（廣泛蒐集多個受害者 PII）研究相對不足，但實務上更具威脅性，因為攻擊者可從公開來源輕易取得初始少量 PII 作為種子。

**論文動機**：開發一種**無需 fine-tuning 或 jailbreak**、基於**增強式 Few-Shot Learning** 的直接查詢方法，利用 LLMs 的「記憶化（memorization）」與「關聯（association）」能力，從少量公開 PII 種子中高效提取大量真實（in-training）PII。Few-Shot Learning 類似人類從少數範例快速學習，適合此情境。

研究區分兩種攻擊：
- **Non-targeted**：針對特定職業（如律師、會計師、醫生、記者）廣泛提取。
- **Targeted**：針對特定個人，使用部分已知資訊提取其餘 PII。

---

### 2. 主要方法與成果

#### Non-targeted Few-Shot 提取

- **Online Learning-based Few-Shot Example Selection**：初始從網路上抓取特定職業的公開 PII 三元組（name, email, phone）作為候選池。將選擇視為 online learning 問題，使用 embedding 特徵 + 品質標籤（if_in_training、hit_rate）計算優先級，混合 greedy/weighted random 選擇策略。每次查詢後驗證新暴露 PII（Google 搜尋 + LLM 輔助），將真實 in-training PII 加入池中、移除非訓練 PII，並位置加權更新權重（prompt 後段例子影響更大）。

- 分**初始階段**（短 few-shot，避免非訓練 PII 過多干擾）與**最終階段**（長 few-shot，利用純 in-training PII 提升記憶化）。

**成果**：在 4 個 LLM（GPT-3.5/4/4o、Claude-3.5）上，8000 查詢提取 3912 個真實 PII 三元組，**攻擊成功率 (ASR) 48.9%**，每提取一個 PII 約 2 次查詢，**成本僅 $0.012**。GPT-4o 最具性價比（ASR 65.6%）。

#### Targeted Few-Shot 提取

- **Query Augmentation through Prompt Chaining**：對目標個人與 few-shot 例子，使用 LLM 生成額外描述（description）、email domain、phone area code 等輔助資訊，串聯成豐富提示，提升關聯能力。

**成果**：在 Enron 等資料集上，超越 SOTA 方法 10%–60% ASR 提升。例如 email 提取從 baseline ~22% 提升至 50–81%。跨 The Pile、CC-News 等資料集亦展現良好泛化性。電話號碼提取較難，但仍有顯著改善。

**整體貢獻**：低成本、大規模真實 PII 提取；揭示部分個人資料外洩即可導致大規模隱私 breach；提供 code（Zenodo）。

---

### 3. 分析與洞見

- **Few-Shot 優化關鍵**：隨機選擇不穩定；online learning + in-training PII 替換 + 位置加權反饋大幅提升效能（ablation 研究證實各組件必要性）。長 few-shot 在 in-training 例子下更有效，反之短 few-shot 較佳（Finding I）。

- **PII 來源洞察**：提取的 7919 個 PII 來自 65 類網站，**Consumer Information（22.7%）** 是重大隱私 breach 來源；Business、政府/軍事、教育等亦佔比高。LLM 聚合放大風險，即使公開資料亦可被惡意利用。

- **模型間比較**：GPT-4o 最易提取（規模與 context window 影響）；GPT-4o 與 GPT-4 訓練資料相似度高（PII 重疊多）。

- **邊緣考量**：驗證使用公開網頁，可能有 false negative（已下架資料）；論文額外用 Internet Archive/Common Crawl 二次驗證，證實方法能恢復部分「已消失」PII。防禦評估顯示現有 model editing（如 REVS）與 query-time（如 PAPILLON）防禦僅部分有效，ASR 仍高且有 overhead。

- **倫理與實務意涵**：凸顯 LLM 對齊不足；攻擊者僅需公開種子 PII 即可大規模操作；對隱私法規、資料清洗、differential privacy 等提出挑戰。

**潛在限制**：依賴 API 成本與速率限制；驗證依賴搜尋引擎；對高度防護或未公開 PII 效果未知。未來可探索更多 PII 類型（如地址、密碼）或跨模型轉移。

---

### 4. 結論

論文提出一套實用、高效的**增強式 Few-Shot Learning 框架**，大幅提升 LLM PII 提取能力，無需破壞對齊或 fine-tuning，即可在低成本下實現大規模 non-targeted 與高精準 targeted 攻擊。這不僅量化了 LLM 隱私風險的嚴重性（數千真實 PII、跨職業/資料集），也揭示訓練資料聚合與部分資訊洩露的連鎖效應，為 LLM 安全防護提供重要參考。

---

**文章連結**：

- **PDF 下載**：https://www.usenix.org/system/files/usenixsecurity25-cheng-shuai.pdf
- **會議頁面**：https://www.usenix.org/conference/usenixsecurity25/presentation/cheng-shuai
