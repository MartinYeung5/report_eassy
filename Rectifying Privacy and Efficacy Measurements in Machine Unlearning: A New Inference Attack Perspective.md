# Rectifying Privacy and Efficacy Measurements in Machine Unlearning: A New Inference Attack Perspective (N. Naderloui et al., USENIX Security 2025)
# 修正機器遺忘中的隱私與效能測量：基於新推論攻擊視角的分析框架（RULI）



### 核心問題與動機

機器遺忘（Machine Unlearning）旨在**高效從已訓練模型中移除特定資料**（忘記集 \( D_f \)），以符合隱私法規（如 GDPR 的「被遺忘權」）、修正有害內容或適應資料變化。

精確遺忘（從頭重訓排除 \( D_f \)）雖理論完美，但對大模型而言極不切實際。因此，**inexact unlearning 方法**（如 Scrub、GA/GA+、NegGrad+、ℓ1-Sparse 等）成為主流，透過修改模型權重或蒸餾等方式來近似移除影響。

### 現有評估框架的關鍵缺陷（Pitfalls）

1. **平均情況（Average-case）主導**：多數工作使用整體資料集的聚合指標（如平均準確率或 population MIA），忽略個別樣本的 memorization 差異。許多樣本本就「安全」（不易被 MIA 攻擊），導致嚴重低估高風險樣本的隱私洩露風險。

2. **隨機樣本目標**：評估時多使用隨機或單類別樣本，未針對易受攻擊的 **vulnerable samples**（高 memorization 樣本），無法揭示真實漏洞。

3. **效能（Efficacy）測量不足**：常僅比對 unlearned model \( \theta_U \) 與 retrained model \( \theta_R \) 的整體準確率（Accuracy on \( D_r \)、\( D_f \)、test set），但這無法捕捉 **per-sample** 行為差異，也無法有效區分「隱私」（是否洩露忘記樣本存在）與「效能」（是否真正近似重訓移除影響）。

### 動機

作者受 Hayes et al. 等工作啟發，提出需要更強的 **sample-level 攻擊** 來驗證 inexact 方法。**RULI 框架**正是為了解決上述問題，提供基於 game-based 的嚴謹評估基礎，同時衡量隱私洩露與效能，推動更可靠的 unlearning 設計。

---

## 結果/成果：RULI 框架與實驗表現

### RULI（Rectified Unlearning Evaluation Framework via Likelihood Inference）

這是本文的核心貢獻：

- **雙目標攻擊**：基於 **Likelihood Ratio Test（LRT）** 和 **Kernel Density Estimation（KDE）**，利用 shadow models 建構多種分佈（In/Out/Unlearned/Held-out 等）。

- **Privacy（Game 2）**：比較 unlearned model 輸出與 held-out 分佈，判斷目標樣本是否曾被訓練並遺忘。

- **Efficacy（Game 3）**：引入 Test model \( \theta_T \)（依樣本狀態切換 unlearned/retrained 輸出），通過比較評估是否真正近似重訓。

### 針對性設計
- 使用 **Canary injection** 技術，將 vulnerable samples（先用 LiRA 識別的高 memorization 樣本）注入忘記集，模擬真實高風險情境。
- 支援多種任務：影像分類（CIFAR-10/100、TinyImageNet + ViT）與文字生成（WikiText-103 + GPT-2）。

### 主要實驗成果（CIFAR-10 等基準）

- **隱私洩露**：RULI 在 vulnerable + protected 混合設定下大幅優於平均情況攻擊與 U-LiRA。例如 GA+ 下 **TPR@1% FPR 可達 20%+**，而平均情況攻擊常低估數倍至十倍。ℓ1-Sparse 相對穩健，但代價是整體 memorization 降低。

- **效能**：多數 inexact 方法與重訓模型存在顯著可區分性（Attack ACC 常 >60–70%），證明難以完美近似。unlearning 還會意外損害剩餘 vulnerable samples 的 memorization（準確率大幅下降）。

- **通用性**：在 ViT + TinyImageNet 以及語言模型上同樣有效，文字 7-gram unlearning **TPR@1% FPR 高達 54%**。

- **效率**：Shadow model 訓練成本合理（相較 U-LiRA 更低），支援並行處理多目標樣本。

**實作開源於 GitHub**，包含完整攻擊 pipeline 與範例程式碼。

---

## 分析與洞見

- **隱私 vs. 效能區分**：兩者密切相關但本質不同。強隱私保護不保證高效能，反之亦然。僅靠平均準確率比對無法捕捉 per-sample 的細微差異，這是現有評估的重大盲點。

- **Vulnerable Samples 的重要性**：unlearning 在 batch 平均梯度更新下，對高 memorization 樣本的效果較差；混合 protected samples 時更難完全移除。**Canary injection** 是有效的壓力測試方式。

- **方法特徵**：
  - Gradient-based 方法（如 GA+、NegGrad+）在效能上較弱，但隱私洩露明顯。
  - Sparse 方法較穩健，但會犧牲模型容量與剩餘資料的 memorization。

- **對領域的影響**：強調 unlearning 評估需從 average-case 轉向 **targeted per-sample** 評估，類似現代 MIA 研究趨勢（LiRA 等）。這有助於未來設計更 robust 的演算法，也提醒實際部署時需採取更保守的隱私保證。

- **邊緣考量與限制**：
  - 攻擊假設 black-box 存取最終 unlearned model（符合現實情境），attacker 可知訓練/遺忘演算法並建構 shadow models。
  - 對 certified unlearning 或大規模 LM 的延伸仍有研究空間。
  - 潛在限制包括計算成本（雖已優於部分 baseline）與特定超參數調校的依賴性。

---

## 結論

本文透過嚴謹的 **game-based 框架** 與新型 inference attack（RULI），成功修正了機器遺忘評估中的核心缺陷，揭示現有 SOTA 方法在隱私與效能上的顯著不足。實驗充分證明 inexact unlearning 難以同時達成高效移除與強隱私保護，尤其在高風險樣本上。

### 貢獻與啟示
**RULI** 提供了一個可擴展、細粒度的評估工具，為 unlearning 研究奠定更可靠的基準，推動從「聲稱移除」走向「可驗證移除」。

**未來方向**包括：
- tighter privacy bounds
- certified 方法整合
- 大模型與多模態任務延伸

這不僅是重要的技術進展，更是對 AI 隱私合規與可信部署的實務貢獻，**強烈推薦研究者與工程師深入參考與應用**。

---

**文章連結：**

- **arXiv**：https://arxiv.org/abs/2506.13009  
- **PDF**：https://arxiv.org/pdf/2506.13009.pdf  
- **USENIX Security 2025 官方版本**：https://www.usenix.org/system/files/usenixsecurity25-naderloui.pdf


