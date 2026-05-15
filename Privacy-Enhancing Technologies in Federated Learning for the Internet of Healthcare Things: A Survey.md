# Privacy-Enhancing Technologies in Federated Learning for the Internet of Healthcare Things: A Survey (F. Mosaiyebzadeh et al., arXiv:2303.14544, 2023)
# 元宇宙應用之互動式快速計算卸載與資源分配聯合優化方法

---

**論文摘要**  
這是一篇針對「醫療物聯網（Internet of Healthcare Things, IoHT）」情境下，聯邦學習（Federated Learning, FL）結合隱私增強技術（Privacy-Enhancing Technologies, PETs）的系統性調查研究。論文填補了先前文獻中「缺乏從 PETs 視角全面檢視 FL 在 IoHT 應用」的空白，針對敏感醫療資料在分散式環境下的隱私洩露風險，提出完整的分類框架、應用案例與挑戰分析。

---

## 核心問題與動機

IoHT 透過穿戴式醫療裝置（如智慧手錶、血壓監測器、心率追蹤器）即時收集生理指標，結合 AI 模型實現疾病預防、診斷、監測與治療。然而，傳統集中式機器學習需將大量原始醫療資料上傳中央伺服器，面臨嚴重隱私與安全危機：

- **資料敏感性**：健康資料屬於高度個人化資訊，一旦洩露可能導致身分識別、歧視或詐欺。  
- **法規限制**：HIPAA（美國）、GDPR（歐盟）等嚴格要求資料最小化原則與患者同意，醫院或裝置間資料共享幾乎不可能，導致資料孤島（data silos）與模型偏差。  
- **FL 本身的不足**：雖然 FL 讓裝置或醫院在本地訓練模型、僅交換參數更新（gradient/model updates），但仍存在多種攻擊向量，包括梯度反演攻擊（gradient leakage）、模型中毒（poisoning）、推論攻擊（inference attacks）與後門攻擊（backdoor attacks）。這些攻擊在 IoHT 即時、低功耗裝置上特別危險，可能讓攻擊者從共享更新中重建患者資料。

論文動機明確指出：單純 FL 無法提供形式化隱私保證（formal privacy guarantees），因此需引入 PETs 強化保護。調查目標是回答「PETs 如何與 FL 結合以滿足 IoHT 隱私需求？」「現有整合的優缺點為何？」「剩餘挑戰與未來方向」。  

這不只是技術問題，更是跨領域議題，涉及醫療倫理、資料治理、邊緣運算資源限制，以及後疫情時代對分散式醫療 AI 的迫切需求。論文強調，若無 PETs，FL 在 IoHT 的實際部署將難以符合法規與患者信任，阻礙智慧醫療的全面發展。

## 研究成果

論文系統回顧 19 篇核心研究，提出**四類 PETs 分類框架**（Anonymization、Cryptographic、Perturbation、Blockchain），並以表格形式比較其在 FL-IoHT 中的應用、資料集、目標攻擊、開源狀態與效能指標。

### 1. Anonymization（匿名化）
- 包含 k-anonymity、l-diversity、t-closeness 等。  
- 應用於本地資料預處理（如 MIMIC-III、Pima Indians 糖尿病資料集、心臟病 Cleveland 資料集）。  
- **成果**：可維持高準確度且計算開銷低，但資訊遺失（information loss）明顯，對 linkage attacks 較脆弱。

### 2. Cryptographic（密碼學）
- 以同態加密（Homomorphic Encryption, HE，如 CKKS）、安全多方計算（Secure Multi-Party Computation, SMC）與零知識證明（ZKP）為主。  
- 代表案例：HAM10000 皮膚癌檢測、UP-FALL 老人跌倒偵測、3D 腦部 MRI 分析。  
- **成果**：提供強大隱私保證，能在水平／垂直 FL 中安全聚合參數；但計算與通訊開銷極高，適合資源較充足的醫院情境。

### 3. Perturbation（擾動）
- 主要為差分隱私（Differential Privacy, DP），分為 Global DP 與 Local DP。  
- 應用案例：EHR 醫院死亡率預測（FL-SIGN-DP 降低頻寬）、ADReSS 阿茲海默症檢測、COVID-19 胸部影像辨識。  
- **成果**：有效抵禦推論攻擊，部分方法（如 DP 結合簽名）可顯著節省頻寬，但模型準確度隨噪音增加而下降。

### 4. Blockchain（區塊鏈）
- 提供不可變性與去中心化聚合（如 FedMedChain 用於 COVID-19 預測）。  
- **成果**：提升可追溯性與 GDPR 相容性，但公開地址可能造成客戶端共謀風險。

論文另以圖表呈現攻擊類型（poisoning、inference、backdoor），並指出多數研究使用公開醫療資料集（如 HAM10000、ADReSS、DarkCOVID），少數提供開源程式碼。整體而言，PETs 能讓 FL 在 IoHT 中同時滿足隱私與實用性，但目前尚無單一技術能完美兼顧所有需求，**混合方法（如 DP+HE）** 已逐漸成為主流趨勢。

## 分析與洞見

論文不僅羅列技術，更深入剖析權衡取捨與系統性洞見：

- **隱私 vs. 實用性（Utility）權衡**：DP 增加噪音可強化隱私，卻直接損害模型準確度，尤其在 IoHT 低品質、小樣本資料上更明顯；HE 提供加密運算但延長訓練時間，在即時監測（如心率異常警報）情境下可能造成延遲。  
  **邊緣案例**：資源受限的穿戴裝置（電池、運算力）下，SMC 或重度 HE 幾乎不可行，需改用輕量 Local DP。

- **攻擊面與威脅模型**：FL 雖不傳原始資料，但共享更新仍可被惡意伺服器或客戶端推斷。k-anonymity 在跨機構情境易受 linkage attacks；區塊鏈雖去中心化，卻引入共謀風險（collusion）。  
  **關鍵洞見**：IoHT 的異質性（heterogeneity）——裝置硬體、資料分布、網路條件差異——放大這些問題，傳統集中式 PETs 無法直接套用。

- **趨勢與缺口**：後疫情研究明顯增加 COVID-19 相關應用；混合 PETs 與聯邦轉移學習（Federated Transfer Learning）是新方向。但仍存在缺口：缺乏 IoHT 專屬大型真實資料集、形式化隱私證明不足、對垂直 FL 與跨國情境討論偏少。

- **更廣泛意涵**：在法規日趨嚴格的時代，PETs 讓 FL 成為合規的「隱私優先」解決方案，同時降低資料傳輸成本、提升模型泛化性（減少偏差）。然而，若忽略邊緣情境（如離線裝置、惡意參與者），可能導致假性安全（false security），反而阻礙採用。論文呼籲未來研究應聚焦輕量混合架構、可量化隱私-效能指標，以及與 5G/6G 邊緣運算的整合。

## 結論與未來展望

論文總結指出，**PETs 是讓 FL 在 IoHT 真正可行的關鍵**，使分散式醫療 AI 能在不犧牲患者隱私的前提下實現跨機構協作。四大類技術各有適用場景：
- 匿名化適合快速原型  
- 密碼學提供最強保證  
- 擾動兼顧效率  
- 區塊鏈強化可信度

**最終推薦**：根據具體 IoHT 應用（即時監測 vs. 長期預測、資源豐富 vs. 受限）選擇或混合 PETs，並優先處理計算開銷、攻擊韌性與可擴展性等開放挑戰。

作者強調，這不僅是技術調查，更是對未來智慧醫療生態的呼籲——唯有將隱私內建於設計（privacy-by-design），才能讓 IoHT 真正惠及全民，同時符合倫理與法規。論文的最後洞見是：雖然目前仍存在效能瓶頸，但 PETs 與 FL 的結合已為下一代隱私保護醫療系統奠定堅實基礎，值得研究社群與產業共同投入。

---

** 論文：
IEEE Xplore: https://ieeexplore.ieee.org/abstract/document/11434888 (DOI: 10.1109/TSC.2026.3674125)
