Optimized Privacy-Preserving Federated Recommender Systems (T. Ganesan et al., 2025)
基於橢圓曲線加密全同態與局部差分隱私的優化隱私保護聯邦推薦系統分析與實作總結


### 🔍 核心問題與動機

傳統推薦系統高度依賴集中式收集用戶原始互動資料（評分、點擊、行為），極易造成嚴重隱私外洩，包括資料洩露、第三方濫用，以及違反 GDPR 等隱私法規。

雖然**聯邦學習**讓用戶端本地訓練模型、僅上傳梯度或參數，避免直接分享原始資料，但仍存在以下挑戰：

- **隱私攻擊**：梯度或模型參數可能被逆向工程，推斷出用戶敏感資訊。
- **安全威脅**：通訊過程易遭竊聽、篡改或中間人攻擊。
- **效能開銷**：加密機制帶來高計算與通訊成本，影響推薦準確度與系統可擴展性。
- **實務限制**：分散式環境中，用戶端裝置資源有限，需在**隱私保護（utility-privacy trade-off）** 與推薦品質間取得平衡。

**論文動機**：填補上述研究缺口，提出 **Optimized-FedRec** 框架，結合 **Elliptic Curve Cryptography (ECC) 基於的全同態加密 (FHE)** 與 **Local Differential Privacy (LDP)**，打造高效、安全且真正可部署的隱私保護聯邦推薦系統。特別適合電影推薦、電子商務等大規模分散式應用場景。

---

### 📊 實驗結果與主要成果

論文使用 **MovieLens 100K** 與 **MovieLens 1M** 兩個標準基準資料集進行全面實驗，評估指標包含：

- 推薦準確度（MAE、RMSE）
- 加密/解密時間
- 計算與通訊成本
- 不同隱私預算（ε）下的效能表現

**核心成果**：

- **推薦效能**：在 ε=1.6 時，MovieLens 100K 的 MAE 降至 **1.023**、RMSE **33**；MovieLens 1M 的 MAE **0.501**、RMSE **29**，均優於傳統 DP 與其他基線方法，在強隱私保護下仍維持高推薦品質。
- **加密效率**：ECC-FHE 機制在加密/解密時間、計算成本與通訊開銷上大幅優於 Paillier 等傳統同態加密方案（顯著降低毫秒級時間與位元組傳輸量）。
- **隱私保護**：採用 Laplace 機制在用戶端對梯度注入噪聲，結合 HMAC 確保通訊匿名性與完整性。在 ε 範圍 0.1~1.6 下，Proposed LDP 的 MAE/RMSE 均優於標準 DP，提供更強保護。
- **整體優勢**：在隱私、準確度與效率間取得優異平衡，證明框架具高度實作可行性，適合部署於資源受限的邊緣裝置。

---

### 💡 技術分析與洞見

#### 主要技術創新
1. **ECC + FHE 核心**：利用橢圓曲線輕量特性實現全同態加密，允許伺服器在密文狀態下直接聚合梯度，無需解密。相較 Paillier，大幅降低金鑰大小與計算複雜度，適合行動裝置與 IoT 環境。採用雙重加密流程進一步強化安全性。
2. **LDP 整合**：用戶端對梯度添加 Laplace 噪聲，實現真正的本地化差分隱私，中央伺服器無法看到原始資料。噪聲強度由 ε 參數控制，提供靈活的 privacy-utility 平衡。
3. **HMAC 認證**：確保通訊完整性與匿名性，防範中間人攻擊。
4. **系統架構**：形成完整閉環 — 用戶端（本地訓練 + 擾動 + 加密）→ 伺服器（密文聚合）→ 全局模型更新。

#### 優勢與權衡
- **優勢**：低開銷、高安全性、可相容 FedMF、NCF 等現有聯邦推薦框架。
- **潛在限制**：極低 ε 時噪聲可能略微影響準確度；全同態計算雖已優化，仍建議搭配硬體加速；論文未深入探討非 IID 資料分佈與模型反演等對抗攻擊。
- **邊緣案例**：冷啟動用戶、稀疏資料集下噪聲影響較大；跨裝置計算能力異質性需額外適配機制。

#### 更廣泛洞見
此論文反映當前 AI 隱私研究主流趨勢 — **混合加密 + 差分隱私機制**。相較純聯邦或純加密方案，Optimized-FedRec 提供更務實的「優化」平衡，可延伸應用至 IoT、醫療等領域。

---

### 🛠️ 專案實作建議（推薦 GitHub 開發方向）

- **可複現性**：使用 MovieLens 資料集 + PyTorch / Flower 聯邦框架，整合 PyECC、TenSEAL 實作 ECC-FHE 與 Laplace 噪聲。
- **擴展方向**：
  - 整合 Graph Neural Network 或 Large Language Model 推薦
  - 真實世界延遲與能源消耗測試
  - 加入攻擊模擬模組（模型反演、成員推斷等）
- **評估框架**：新增 NDCG、Precision@K、Recall@K 等排名指標

---

### 📝 結論

**Optimized-FedRec** 提出了一個創新且實用的解決方案，有效解決聯邦推薦系統中的隱私洩露、計算開銷與安全風險。透過 ECC-FHE 與 LDP 的巧妙結合，不僅在理論上證明可行性，更以嚴謹實驗展示優異效能。

此框架為隱私優先推薦系統領域提供重要參考藍圖，強調 **「隱私即設計」（Privacy by Design）** 的核心價值。未來可朝量子安全加密、聯邦持續學習、大規模真實部署驗證及跨域應用繼續深化。

---

### 📚 論文連結

- **ResearchGate (PDF 下載)**: [https://www.researchgate.net/publication/391691325_Optimized-FedRec_Optimized_Privacy-Preserving_Federated_Recommender_System_with_Elliptic_Curve_Cryptography_based_Homomorphic_Encryption_and_Local_Differential_Privacy](https://www.researchgate.net/publication/391691325_Optimized-FedRec_Optimized_Privacy-Preserving_Federated_Recommender_System_with_Elliptic_Curve_Cryptography_based_Homomorphic_Encryption_and_Local_Differential_Privacy)
- **ScienceDirect**: [https://www.sciencedirect.com/science/article/pii/S1877050925012165](https://www.sciencedirect.com/science/article/pii/S1877050925012165)  
  (DOI: 10.1016/j.procs.2025.04.115)
- **期刊**: *Procedia Computer Science* 259 (2025) 1602–1611，FTNCT 2025 會議論文

