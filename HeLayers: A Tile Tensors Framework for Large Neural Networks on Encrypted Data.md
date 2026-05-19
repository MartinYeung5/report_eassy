HeLayers: A Tile Tensors Framework for Large Neural Networks on Encrypted Data (Ehud Aharoni et al., PETS 2023)
基於 Tile Tensors 的大規模神經網路加密資料框架

---

## 📌 1. 核心問題與動機

同態加密允許在**不解密**的情況下對資料進行計算，為醫療、金融、Web3 等隱私敏感場景提供了強大保障，符合 GDPR、HIPAA 等法規要求。

### 主要挑戰
- HE 方案（如 CKKS）採用 SIMD 運算，一個密文僅能容納固定數量的 **slot**（由安全參數決定）。
- 高維張量（影像、批次資料）需進行複雜的 **packing**，不同操作（卷積、矩陣乘法）對 packing 形狀需求差異極大。
- 不佳的 packing 會導致：
  - 記憶體浪費（slot 未充分利用）
  - 大量旋轉（rotation）操作，增加延遲與計算成本
  - 通訊開銷上升
- 現有非互動式（non-client-aided）方案在大型網路（如 224×224×3 的 AlexNet）上效能極差，最佳實現需 **超過 3 小時**（80-bit 安全），遠無法實用。
- client-aided 方法雖快，但需用戶全程線上，存在模型萃取風險，且違背完全外包的初衷。

### 研究動機
開發一個 **packing-oblivious**（打包無感知）的框架，讓開發者只需專注神經網路設計，框架自動處理所有底層 packing 優化。同時聚焦非互動式、HE-friendly 模型（以多項式激活取代 ReLU/MaxPool），在準確度與效能間取得平衡。

---

## 🚀 2. 主要成果

- **Tile Tensors 資料結構**  
  將張量拆分成固定形狀的 **Tile**（每個 Tile 對應一個密文），提供類似 PyTorch 的標準 Tensor API（加法、乘法、sum、broadcasting、卷積等）。內部自動處理 packing、旋轉與 HE 操作，支援多種 packing 策略（interleaved、rotated wide tensors 等）。

- **Packing Optimizer**  
  自動探索多種 packing 配置，估計時間與記憶體成本，根據使用者目標（低延遲 / 高吞吐 / 低記憶體）選擇最佳方案。提供描述 packing 的專用語言，便於設計與重現。

- **新型 2D Convolution 演算法**  
  針對大型輸入提出高效 packing 與實現，可連續執行多層卷積，無需昂貴的 im2col 預處理（避免增加乘法深度），特別適合 AlexNet 等深層網路。

- **實驗成果**
  - CryptoNets 基準：不同 packing 下延遲從 **0.86s 提升至 11.1s** 不等，展示優化效果顯著。
  - **HE-friendly AlexNet**（224×224×3 輸入）：完整 inference 僅需 **約 5 分鐘**（128-bit 安全，無 bootstrapping），比先前非互動式方案快數個數量級，是當時最大且最快的實現。
  - 框架已開放 **HELayers SDK**（非商業使用），被後續多篇 PPML 工作廣泛引用。

---

## 🔍 3. 分析與洞見

### 優點與創新
- **極高易用性**：開發者可像寫普通深度學習程式一樣操作 Tensor，框架自動處理所有 HE 細節，大幅降低開發門檻。
- **高度彈性**：Tile 設計天然支援多維度、批次處理與長序列，特別適合 CNN。
- **安全與實用平衡**：堅持 non-client-aided 模式，使用者只需加密一次資料即可離線，隱私保護更完整。
- 為後續 FHE 編譯器與 PPML 框架（如 Orbit、HEPack）奠定重要基礎。

### 限制與注意事項
- 需使用多項式逼近激活函數，可能略微影響模型準確度。
- 目前未整合 bootstrapping，限制電路深度，適合 HE-friendly 架構。
- 對 Transformer 等 Attention-heavy 模型的支援需額外驗證。
- 效能仍高度依賴底層 HE 函式庫（HELib、SEAL），未來 GPU / 硬體加速空間很大。
- Packing Optimizer 在極大型網路上的編譯時間可能較長。

### 更廣泛意義
HeLayers 代表同態加密從理論走向實用的關鍵一步。在 **Web3、聯邦學習、醫療 AI、隱私計算** 等領域具備巨大潛力，讓用戶能安全地將加密資料上傳雲端執行大型模型，真正實現「隱私即預設」（Privacy-by-Design）。

---

## 🏁 4. 結論

HeLayers 透過 **Tile Tensors** 與自動 Packing Optimizer，成功將大型神經網路在加密資料上的非互動式推論推向實用層級（AlexNet 5 分鐘），大幅縮小 HE 與明文計算的效能差距。

其核心貢獻在於提供**直觀抽象、通用 packing 機制與高效卷積實現**，讓隱私保護 AI 開發者得以專注高層邏輯而非底層密碼學細節。

隨著 HE 硬體加速、更好 HE-friendly 模型以及編譯技術的進展，**完全實用的加密 AI 時代即將到來**。

本專案適合作為學習與二次開發的起點：  
- 基於開放的 HElayers SDK 實作 Demo  
- 擴展支援更多網路架構  
- 整合 bootstrapping、GPU 加速或 Web3 應用

---

## 📚 論文連結

- **PETS 2023 正式版本**：https://petsymposium.org/popets/2023/popets-2023-0020.pdf  
- **arXiv 版本**：https://arxiv.org/pdf/2011.01805.pdf  
- **IBM Research 頁面**：https://research.ibm.com/publications/helayers-a-tile-tensors-framework-for-large-neural-networks-on-encrypted-data
