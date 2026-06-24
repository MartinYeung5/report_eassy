# LLM Unlearning Benchmarks are Weak Measures of Progress
# LLM Unlearning Benchmarks 是進展的薄弱衡量指標：CMU 論文深度分析

### 核心問題與動機

機器學習中的「Unlearning」（遺忘/抹除）旨在讓模型在訓練後移除特定資料的影響，而無需從頭重新訓練全部資料。這在 LLM（大型語言模型）中特別重要，因為涉及隱私保護（例如移除敏感個人資料）、安全（移除有害知識）以及法規合規（如 GDPR 的「被遺忘權」）。然而，LLM 規模龐大，完整重新訓練不切實際，因此研究社群轉向**近似 unlearning 方法**，並依賴**經驗基準（empirical benchmarks）**來評估成效。

**主要問題**：現有 LLM unlearning 基準（如 TOFU、WMDP、Who’s Harry Potter?、TDEC、PKU-SafeRLHF 等）普遍過於樂觀且具誤導性。它們通常將評估拆分成兩個獨立部分：
- **Forget Set**：測量是否成功遺忘特定知識（準確率應降低）。
- **Retain Set**：測量是否保留其他無關知識（準確率應維持）。

作者（Pratiksha Thaker 等，CMU）透過廣泛實驗發現，這些基準忽略了現實查詢中 **forget 與 retain 資料之間的依賴關係**，也容易鼓勵方法過擬合測試查詢本身，而非真正解決 unlearning 問題。

**動機**：作者調查了 2024 年 72 篇 unlearning 論文，發現 82% 使用 forget/retain 結構，前五大基準佔近半數評估和 80% 引用。這使得基準成為社群進展的關鍵驅動因素，但若基準本身薄弱，將誤導整個領域。論文強調，即使沒有統一的形式化定義，至少應確保基準符合高層直覺：unlearning 應在真實、多樣查詢下有效，而非僅在特定測試集上表現良好。

這反映了更廣泛的 LLM 評估挑戰（基準脆弱性），但 unlearning 在隱私關鍵情境下風險更高——錯誤的「成功」可能導致實際隱私洩露。

### 結果/成果

作者對多個流行基準進行**簡單、非對抗性修改**，揭示了現有方法的失效：
1. **Forget-Retain 依賴性漏洞（Section IV）**：
   - **TOFU**：將 forget 作者與 retain 作者的問題合併詢問。許多 unlearning 方法（如基於 DPO 的偏好優化、ECO）在單獨 retain 查詢時表現良好，但在組合查詢時要麼拒答（破壞 retain 效用），要麼錯誤處理兩者。Gradient Ascent 較穩定但整體分數較低。
   - **WMDP**：在 retain 集的多選題中，將一個錯誤選項替換為 forget 相關關鍵詞（如 “SARS-CoV-2”）。RMU 等方法 retain 準確率大幅崩潰（接近隨機），甚至比未 unlearning 的基底模型更脆弱。

2. **過擬合測試集（Section V）**：
   - **TOFU**：簡單關鍵字過濾（搜尋 forget 作者姓名）即可完美通過基準，但這在現實中難以泛化。
   - **WMDP**：ECO 方法的分類器過擬合提示中的 spurious feature（如 “college” 關鍵字），移除後表現崩潰。
   - **PKU-SafeRLHF**：有些工作直接在測試集上訓練，缺乏 held-out 集。
   - 改變查詢類型（e.g., 多選改成開放式）也容易重新引出已「遺忘」資訊。

這些修改暴露了基準的樂觀偏差：方法看似成功，但面對輕微真實世界變異即失效。作者也討論了 forget 集定義不明確的基準（如 RWKU），導致評估模糊。

### 分析與洞見

**多角度分析**：
- **基準設計根本缺陷**：Forget/Retain 分離假設兩者完全獨立，但現實查詢常有交叉依賴（e.g., 同時問 forget 與 retain 實體）。這鼓勵「分類器式」解決方案，而非真正移除影響。
- **過擬合風險**：基準鼓勵 pre/post-processing 過濾或直接針對測試查詢優化，而非泛化 unlearning。無 held-out 集或多樣查詢格式加劇此問題。
- **與一般 LLM 評估的差異**：一般基準脆弱性主要影響可靠性；在 unlearning 中，則可能直接違反隱私或安全目標。Unlearning 還涉及額外複雜性，如威脅模型不明確、forget 資料是否必須來自訓練集等。
- **邊緣案例與細微差別**：某些方法在特定基準上穩定（如 Gradient Ascent），但整體效用低；強健性訓練（如 LAT）有時反而增加脆弱性。基準也未充分區分「unlearning 特定訓練資料」 vs. 「一般審查/對齊」。
- **更廣影響**：這類似其他 LLM 基準批判（e.g., 查詢翻譯、格式變化），但在隱私領域後果更嚴重。社群壓力（快速發表、排行榜）進一步放大問題。

**專案導向洞見**（適合 GitHub 專案應用）：
- 在開發 unlearning 工具時，勿僅依賴單一基準；需自建 perturbation 測試（如組合查詢、關鍵詞替換、格式變更）。
- 考慮實作 membership inference attacks 作為額外指標，或聚焦 finetuning 資料的 exact unlearning 以建立可靠 baseline。
- 專案可擴展：建立更 robust 的 benchmark 套件，包含 held-out 集、多格式查詢、依賴性測試案例。

### 結論與建議

**主要立場**：現有 LLM unlearning 基準在最佳情況下是有限的進展衡量，在最壞情況下具誤導性。社群應謹慎解讀基準結果，而非視為可靠證據。

**文章連結**：
- CMU ML Blog：https://blog.ml.cmu.edu/2025/04/18/llm-unlearning-benchmarks-are-weak-measures-of-progress/
- arXiv 論文（Position Paper）：https://arxiv.org/abs/2410.02879（或 PDF：https://arxiv.org/pdf/2410.02879）
