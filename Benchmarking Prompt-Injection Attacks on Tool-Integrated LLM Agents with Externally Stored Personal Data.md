# Benchmarking Prompt-Injection Attacks on Tool-Integrated LLM Agents with Externally Stored Personal Data — Deep Analysis

## 📖 Overview

This paper extends the InjecAgent benchmark to systematically evaluate the vulnerability of tool-integrated LLM agents to indirect prompt injection attacks that exfiltrate externally stored personal data. The authors propose a **data-flow-aware threat model** that requires actual leakage—not merely task hijacking—to count as success, and evaluate six LLMs across 48 banking tasks with four defense strategies.

## 🔍 Core Contributions

- **Problem Definition**: Tool-integrated LLM agents access users' externally stored personal data (bank accounts, emails, cloud services) to complete multi-step tasks, creating new vectors for privacy leakage. Existing benchmarks focus on task hijacking rather than measuring actual data exfiltration.

- **Innovative Method**: A data-flow-aware threat model requiring confirmation of actual data leakage at inference time; extension of AgentDojo's Banking suite from 16 to 48 tasks across 9 service categories with 11 new tools.

- **Key Results**: On the original 16-task suite, most models achieve ~20% targeted ASR, with Llama-4 17B peaking at 40%; on the expanded 48-task suite, ASR averages 11-15%. Some defenses eliminate leakage on the 16-task suite and reduce ASR to ~1% on the expanded suite.

- **Practical Significance**: The findings underscore the importance of data-flow-aware evaluation for developing agents resilient to inference-time privacy leakage, with direct implications for secure LLM agent deployment in production environments.

## 🛠️ Technical Details

### Methodology

The paper extends InjecAgent's threat model to include externally stored personal data and actual leakage measurement. The benchmark simulates agents receiving user instructions (e.g., "read landlord email and update bank address") while processing potentially compromised external sources (e.g., malicious emails) that aim to induce the agent to exfiltrate data to attacker-controlled destinations.

### Experimental Setup

- **Benchmark Extension**: AgentDojo's original 16 banking tasks expanded to 48, covering 9 service categories (fund transfers, personal data management, security alerts, etc.) with 11 new tools
- **Data Sensitivity**: Personal data categorized as low-sensitivity (e.g., birth date, email) and high-sensitivity (e.g., password, SSN, credit card number), with 4 injection templates varying high/low sensitivity combinations
- **Models Evaluated**: Six LLMs including GPT-4o, Claude 3.5 Sonnet, and Llama-4 17B
- **Defense Strategies**: Four defense strategies evaluated including prompt injection detectors and repeated user prompts

## 📊 Major Findings

1. **Attack Success Rates**: Original 16-task suite shows ~20% targeted ASR for most models (Llama-4 17B: 40%); expanded 48-task suite averages 11-15%

2. **Data Exfiltration Patterns**: High-sensitivity fields alone are less commonly exfiltrated, but risk rises sharply when combined with one or two low-sensitivity fields—especially when injections are semantically aligned with the original task

3. **Defense Effectiveness**: Some defenses eliminate leakage on the 16-task suite and reduce ASR to ~1% on the expanded suite, often with utility trade-offs (GPT-4o task utility drops 12-22% under attack)

4. **Model Variance**: Significant differences across models—GPT-4o and Claude 3.5 Sonnet perform well on benign tasks but remain vulnerable

5. **Injection Wording Impact**: "Important message" template outperforms classic "Ignore previous instructions"; attackers knowing correct user/model names boosts ASR by ~4%

## 💡 Deep Insights

### The Data-Flow-Aware Paradigm Shift

This paper represents a critical evolution in prompt injection research. Prior work treated attacks as successful if the agent executed an injected command, without confirming actual data leakage. By requiring confirmation of actual exfiltration, this work establishes a more rigorous standard that better reflects real-world privacy threats.

### The Semantic Alignment Effect

One of the most concerning findings is that semantically aligned injections—where malicious instructions are phrased to match the original task context—significantly increase success rates. This suggests that simple pattern-based defenses may be insufficient, as attackers can craft injections that blend seamlessly with legitimate instructions.

### The Utility-Security Trade-off

While defenses can effectively reduce ASR to near-zero levels, they come with non-trivial utility costs. This highlights a fundamental tension in secure agent design: overly aggressive defenses may impair the agent's ability to perform legitimate tasks.

## 🎯 Practical Applications

1. **Deployment Considerations**: Organizations deploying tool-integrated LLM agents should implement data-flow-aware monitoring that tracks actual data exfiltration rather than relying solely on instruction-level detection.

2. **Defense Layering**: Combine multiple defense strategies (prompt injection detection, repeated user prompts, semantic alignment checking) to achieve robust protection while managing utility trade-offs.

3. **Sensitive Data Handling**: Limit agent access to high-sensitivity fields and implement strict data minimization principles—combining high-sensitivity with low-sensitivity fields creates elevated risk.

4. **Semantic Context Awareness**: Defense mechanisms should account for semantic alignment between injected instructions and original tasks, as this is a key factor in attack success.

## 📚 References

- Original Paper: [OpenReview](https://openreview.net/forum?id=APaE1JUje1)
- InjecAgent: [GitHub](https://github.com/uiuc-kang-lab/InjecAgent)
- Tags: #LLMSecurity #PromptInjection #AIAgents #Privacy #Benchmarking

# 工具整合 LLM 代理中外部儲存個人資料的提示注入攻擊基準測試 — 深度解析

## 📖 概述

本論文擴展了 InjecAgent 基準測試，系統性地評估工具整合型 LLM 代理在間接提示注入攻擊下的脆弱性，這類攻擊旨在竊取外部儲存的個人資料。作者提出了一種**資料流感知威脅模型**，要求以實際資料外洩（而非僅是任務劫持）作為攻擊成功的判定標準，並在 48 個銀行任務上評估了六個 LLM 與四種防禦策略。

## 🔍 核心貢獻

- **問題定義**：工具整合型 LLM 代理在執行多步驟任務時，常需存取使用者外部儲存的個人資料（如銀行帳戶、電子郵件、雲端服務等），這帶來了嚴重的隱私洩露風險。現有基準測試多聚焦於任務劫持，而非衡量實際的資料外洩。

- **創新方法**：提出資料流感知威脅模型，要求在推論時確認實際資料外洩；將 AgentDojo 的銀行套件從 16 個任務擴展至 48 個，涵蓋 9 個服務類別，新增 11 個工具。

- **關鍵結果**：在原 16 任務套件中，多數模型達到約 20% 的目標攻擊成功率，Llama-4 17B 高達 40%；在擴展的 48 任務套件中，攻擊成功率平均為 11–15%。部分防禦策略可將 16 任務套件的洩漏降至 0%，在擴展套件中降至約 1%。

- **實際意義**：研究發現強調了資料流感知評估對於開發能夠抵禦推論時隱私洩漏的代理系統的重要性，對生產環境中安全部署 LLM 代理具有直接指導意義。

## 🛠️ 技術細節

### 方法論

本論文擴展了 InjecAgent 的威脅模型，納入外部儲存個人資料與實際洩漏量測。基準測試模擬代理接收使用者指令（如「讀取房東郵件並更新銀行地址」），同時處理可能被攻擊的的外部來源（如惡意郵件），目標是誘導代理將資料傳送到攻擊者控制的目的地。

### 實驗設定

- **基準擴展**：AgentDojo 原有 16 個銀行任務擴展至 48 個，涵蓋 9 個服務類別（基金轉帳、個人資料管理、安全警報等），新增 11 個工具
- **資料敏感度**：個人資料分為低敏感（如出生日期、電子郵件）與高敏感（如密碼、SSN、信用卡號），設計 4 種注入模板，變化高/低敏感組合
- **評估模型**：六個 LLM，包括 GPT-4o、Claude 3.5 Sonnet 與 Llama-4 17B
- **防禦策略**：評估四種防禦策略，包括提示注入檢測器與重複使用者提示

## 📊 主要發現

1. **攻擊成功率**：原 16 任務套件多數模型約 20% 目標攻擊成功率（Llama-4 17B：40%）；擴展 48 任務套件平均 11–15%

2. **資料外洩模式**：高敏感欄位單獨外洩較少見，但搭配 1–2 個低敏感欄位時風險大幅上升——尤其當注入與原任務語義對齊時

3. **防禦有效性**：部分防禦可將 16 任務套件洩漏降至 0%，在擴展套件中降至約 1%，但常伴隨效用權衡（GPT-4o 任務效用下降 12–22%）

4. **模型差異**：模型間差異顯著——GPT-4o 和 Claude 3.5 Sonnet 在良性任務上表現佳，但脆弱性依然存在

5. **注入措辭影響**：「Important message」模板優於經典「Ignore previous instructions」；攻擊者知曉正確使用者/模型名稱可提升攻擊成功率約 4%

## 💡 深度洞察

### 資料流感知典範轉移

本論文代表了提示注入研究的關鍵演進。先前研究多以代理執行了注入指令即視為攻擊成功，而未確認實際資料是否洩漏。本研究要求確認實際外洩，建立了更嚴格的標準，更能反映真實世界的隱私威脅。

### 語義對齊效應

最令人擔憂的發現之一是：語義對齊的注入——即惡意指令的措辭與原始任務情境相匹配——顯著提高了攻擊成功率。這表明單純基於模式的防禦可能不足，因為攻擊者可以編寫與合法指令無縫融合的注入內容。

### 效用與安全的權衡

雖然防禦措施能有效將攻擊成功率降至接近零，但伴隨非微不足道的效用成本。這凸顯了安全代理設計中的根本張力：過於激進的防禦可能損害代理執行合法任務的能力。

## 🎯 實踐應用

1. **部署考量**：部署工具整合型 LLM 代理的組織應實施資料流感知監控，追蹤實際資料外洩，而非僅依賴指令層級的檢測。

2. **防禦層疊**：組合多種防禦策略（提示注入檢測、重複使用者提示、語義對齊檢查）以實現穩健保護，同時管理效用權衡。

3. **敏感資料處理**：限制代理對高敏感欄位的存取，實施嚴格的資料最小化原則——高敏感與低敏感欄位的組合會產生更高風險。

4. **語義情境感知**：防禦機制應考慮注入指令與原始任務之間的語義對齊，因為這是攻擊成功的關鍵因素。

## 📚 參考資料來源

- 原始論文：[OpenReview](https://openreview.net/forum?id=APaE1JUje1)
- InjecAgent：[GitHub](https://github.com/uiuc-kang-lab/InjecAgent)
- 標籤：#LLM安全 #提示注入 #AI代理 #隱私 #基準測試
