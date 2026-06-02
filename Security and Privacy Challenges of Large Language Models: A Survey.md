# Security and Privacy Challenges of Large Language Models: A Survey (ACM Computing Surveys, 2025)
大型語言模型的安全與隱私挑戰綜述

### 1. 核心問題與動機
大型語言模型（Large Language Models, LLMs）如 GPT 系列，已展現出生成文本、翻譯、問答、程式碼產生等近乎人類水準的能力，並廣泛應用於各領域。然而，隨著模型規模擴大（參數量龐大）及訓練資料來自網際網路的開放來源，安全與隱私漏洞成為重大隱憂。

**主要動機**：
- **訓練資料風險**：LLMs 需要海量網際網路資料，這些資料常含低品質或未經同意的個人資訊，易導致 **Personally Identifiable Information (PII)** 洩露，違反 GDPR、CCPA、HIPAA 等法規。
- **模型本質脆弱性**：LLMs 具備強大生成能力，但也易受操縱，如產生有害內容、記憶訓練資料，或被惡意提示引導。
- **應用部署風險**：在醫療、金融、教育、交通等高敏感領域，安全漏洞可能造成嚴重後果（如誤診、資料外洩、社會工程攻擊）。
- **研究缺口**：早期研究多聚焦效能，安全隱私評估不足。ChatGPT 等模型普及後，jailbreaking、poisoning 等攻擊激增，亟需系統性綜述。

論文動機在於提供及時、全面視角，幫助研究者、開發者與利害關係人理解 LLMs 的優勢與風險，推動安全隱私保護的發展。

### 2. 結果/成果
論文系統分類並分析 LLMs 的安全與隱私攻擊、防御機制、應用風險，並比較既有研究。

**安全攻擊（Security Attacks）主要類型**：
- **Jailbreaking Attacks**：透過精心設計提示（如 DAN "Do Anything Now"）繞過對齊機制，讓模型產生有害、非法或不道德內容。
- **Prompt Injection**：惡意提示覆蓋原始指令，導致模型執行未經授權操作。
- **Backdoor / Data Poisoning Attacks**：在訓練或微調階段植入後門，觸發特定輸入時產生惡意輸出；資料中毒則污染訓練集。
- **其他**：如 Adversarial Attacks、Model Extraction 等。

**隱私攻擊（Privacy Attacks）主要類型**：
- **Membership Inference Attack (MIA)**：推斷特定資料是否用於訓練。
- **Gradient Leakage / Data Reconstruction**：從梯度或模型輸出重建訓練資料。
- **PII Leakage & Memorization**：模型過度記憶訓練資料，導致查詢時意外洩露個人資訊（如姓名、地址、敏感紀錄）。
- **其他**：如 Attribute Inference 等。

**防御機制（Defenses）**：
- **Prompt Engineering**：如系統提示、拒絕機制，但易被進階 jailbreak 繞過。
- **Robust Training**：Differential Privacy (DP)、Adversarial Training、Reinforcement Learning from Human Feedback (RLHF)。
- **Detection & Mitigation**：Gradient Noise、Homomorphic Encryption、Model Stacking、Watermarking。
- **應用層**：內容過濾、監控、聯邦學習（Federated Learning）。

**應用領域風險**：
- 交通（自動駕駛決策操縱）、醫療（診斷誤導、病歷洩露）、教育（作弊、偏見）、治理與科學（假資訊傳播、知識操縱）。

論文強調攻擊演進快速（從手動提示到自動化、黑箱攻擊），防御則常有局限性（如效能損失、無法全面涵蓋新興威脅）。

### 3. 分析與洞見
**多角度分析**：
- **技術層面**：LLMs 的 Transformer 架構與 in-context learning 提升彈性，但也放大攻擊面。規模越大，memorization 越嚴重（scaling law 雙面刃）。
- **社會與倫理層面**：隱私洩露不僅是技術問題，還涉及法規遵守、信任危機與社會不平等（偏見放大）。在民主治理中，假資訊可能影響選舉或公共政策。
- **經濟與實務層面**：企業部署 LLMs 需權衡效能、成本與風險。高敏感產業（如醫療）若未妥善防護，可能面臨巨額罰款或訴訟。
- **邊緣案例與細微差別**：
  - 白箱 vs. 黑箱攻擊：研究者易取得白箱資訊，但真實攻擊多為黑箱。
  - 小模型 vs. 大模型：小型模型較易防護，但效能低；大型模型攻擊面更廣。
  - 動態 vs. 靜態攻擊：攻擊者可持續優化提示，傳統防御難以跟上。
  - 多模態擴展（未來趨勢）：圖文、語音 LLMs 將引入新隱私風險（如影像重建）。

**關鍵洞見**：
- 現有防御多為「patchwork」（拼湊式），缺乏統一框架。Prompt-based 方法簡單但脆弱；訓練時防御（如 DP）雖有效，但會犧牲模型效能。
- 攻擊與防御呈「軍備競賽」：新攻擊（如自動化 PAIR）常擊敗舊防御。
- 跨領域應用放大風險：單一漏洞在醫療或交通中可能導致生命財產損失。
- 研究差距：對先進攻擊的防御不足、評估指標不統一、實際部署情境研究少。

### 4. 結論
論文結論強調，儘管 LLMs 帶來革命性進展，但安全與隱私挑戰若未妥善解決，將嚴重阻礙其可靠部署。作者呼籲：
- 開發更穩健的評估框架與指標。
- 探索新型防御，如結合加密、聯邦學習與可解釋 AI。
- 加強跨學科合作（技術、法規、倫理）。
- 未來方向包括：對抗動態攻擊、隱私增強技術在多模態模型的應用、實際系統級保護，以及平衡效能與安全的 trade-off。

**文章連結**：  
- ACM 正式版本：https://dl.acm.org/doi/10.1145/3712001 (2025 年發表於 ACM Computing Surveys)  
- arXiv 預印本（推薦閱讀完整 PDF）：https://arxiv.org/pdf/2402.00888.pdf (作者：Badhan Chandra Das, M. Hadi Amini, Yanzhao Wu)
