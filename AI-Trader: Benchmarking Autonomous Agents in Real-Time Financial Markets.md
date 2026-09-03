
# AI-Trader: Benchmarking Autonomous Agents in Real-Time Financial Markets

[![arXiv](https://img.shields.io/badge/arXiv-2512.10971-b31b1b.svg)](https://arxiv.org/abs/2512.10971)
[![GitHub](https://img.shields.io/badge/GitHub-HKUDS%2FAI--Trader-blue)](https://github.com/HKUDS/AI-Trader)

**AI-Trader** is the first fully autonomous, real‑time, and contamination‑free benchmark for evaluating Large Language Model (LLM) agents in financial trading. It covers **US stocks, A‑shares, and cryptocurrencies** with live market data. The key finding: *general intelligence does not automatically translate into effective trading ability* — most agents underperform and exhibit weak risk control, which is the decisive factor for cross‑market robustness.

---

## 📌 Core Contributions

- **Problem Definition**  
  Existing LLM benchmarks (e.g., QA, code completion, instruction following) operate in static, deterministic environments and fail to assess agent performance in dynamic, real‑time, high‑uncertainty scenarios. There is no systematic, automated, and contamination‑free framework for evaluating LLM agents in actual financial markets.

- **Innovative Approach**  
  AI‑Trader adopts a **Fully Autonomous Minimal Information Paradigm**:
  - **Zero human intervention** – agents receive only basic context (available tools, current positions, real‑time prices) and must independently search, verify, and synthesise market information.
  - **Three markets** – Nasdaq‑100 (US), SSE‑50 (A‑shares), and top‑10 crypto pairs, with hourly (US) and daily (A‑shares & crypto) trading frequencies.
  - **Observe‑Reason‑Act loop** – agents continuously observe, reason, and act; all information is retrieved via tools, and all decisions are made autonomously under real market constraints.
  - **Tool set** – built on the Model Context Protocol (MCP) with five core tools: *Check Price*, *Search*, *News*, *Math*, and *Trade*.

- **Key Results**  
  Evaluation of six mainstream LLMs (DeepSeek‑v3.1, MiniMax‑M2, Claude‑3.7‑sonnet, GPT‑5, Qwen3‑max, Gemini‑2.5‑flash) over Oct–Nov 2025 reveals:
  - **General intelligence ≠ trading ability** – high performance on conventional benchmarks does not guarantee effective trading.
  - **Risk control determines cross‑market robustness** – it is the most critical factor for consistent performance across different markets.
  - **Liquidity matters** – AI strategies achieve excess returns more easily in liquid markets (US) than in policy‑driven markets (A‑shares).
  - **Limited cross‑market generalisation** – agents struggle to adapt across markets without human guidance.

- **Practical Applications**  
  - Rapid validation of LLM‑based trading strategies before live deployment.
  - Multi‑market and multi‑frequency strategy testing for global asset allocation.
  - Open‑source code and data ([GitHub](https://github.com/HKUDS/AI-Trader)) for community extension.
  - Testbed for validating AI risk‑management models.

---

## ⚙️ Technical Details

### Architecture: Observe‑Reason‑Act Loop

- **Observation Space** – initial market perception consists of current prices **p** = [p₁, p₂, …, pₙ] and portfolio positions **s** = [s₁, s₂, …, sₙ]. Agents can dynamically fetch additional information (fundamental/technical indicators, news, macro data) via tools. The full observation at time *t* is:

  ```
  oₜ = f(p, s, {πᵢ}, i)
  ```

- **Reasoning Process** – follows the ReAct paradigm (“think‑then‑act”). Agents generate intermediate natural‑language reasoning traces (e.g., “Company X’s Q2 earnings beat expectations and its P/E is below industry average, suggesting a buying opportunity”). All traces are recorded for auditability.

- **Action Space** – for each tradable asset, agents can **Buy**, **Sell**, or **Hold**. If an action exceeds available liquidity, the system rejects it and triggers a self‑correction mechanism, forcing the agent to re‑reason and propose a compliant action. The final action is:

  ```
  aₜ = f(oₜ, rₜ)
  ```

  where *rₜ* is the reasoning output.

### Tool Set (MCP‑based)

| Tool | Description |
|------|-------------|
| **Check Price** | Query historical or intraday OHLC prices and volume; auto‑detects market‑specific tickers. |
| **Search** | Retrieve public information (market, company, macro) with strict time‑cutoff to avoid look‑ahead bias. |
| **News** | Provide structured financial news with sentiment signals, timestamps, and related securities. |
| **Math** | Perform necessary numerical calculations during trading. |
| **Trade** | Execute buy/sell orders respecting market rules (e.g., 100‑share lots for A‑shares); updates positions and cash in real time. |

### Evaluation Metrics

- **Cumulative Return (CR)** – total percentage gain/loss over the evaluation period:

  ```
  CR = (∏ₜ₌₁ᵀ (1 + rₜ)) − 1
  ```

- **Sortino Ratio (SR)** – risk‑adjusted return penalising only downside volatility:

  ```
  SR = (r̄ − r_target) / σ_d
  ```

---

## 🧪 Experimental Setup

| Market | Index / Assets | Characteristics |
|--------|---------------|-----------------|
| **US Stocks** | Nasdaq‑100 | High liquidity, transparent information, sensitive to tech and macro factors |
| **A‑shares** | SSE‑50 | Policy‑driven, high retail participation, non‑stationary behaviour |
| **Crypto** | Top‑10 pairs (BTC, ETH, XRP, etc.) | 24/7 trading, extreme volatility, technology‑driven |

- **Trading Frequencies**: Hourly (US) and daily (A‑shares & crypto).
- **LLM Backbones**: DeepSeek‑v3.1, MiniMax‑M2, Claude‑3.7‑sonnet, GPT‑5, Qwen3‑max, Gemini‑2.5‑flash – all wrapped with identical agent structures for fair comparison.
- **Infrastructure**: API‑based LLM calls (no local deployment required); real‑time market data feeds; MCP protocol for tool integration; code available on GitHub.

---

## 🔍 In‑Depth Analysis

**Paradigm shift from static to live evaluation** – AI‑Trader moves LLM assessment from “closed‑book exams” to “live‑fire drills”. Traditional benchmarks test knowledge and reasoning in controlled settings, but real trading exposes agents to incomplete information, time pressure, and real consequences. This reveals a critical gap: *strong performance on static tasks does not predict reliable decision‑making under uncertainty*.

**Why general intelligence fails in trading** – The ability to write poetry or solve math problems does not equate to judging market sentiment, managing risk, or handling timing pressure. These dynamic skills are not captured by conventional benchmarks. Trading demands not just knowledge, but also *respect for uncertainty, sensitivity to risk, and adaptability to time stress*.

**Risk control as the overlooked core** – The paper highlights that risk management is the key to cross‑market robustness. Most models severely lack this ability. Future development of financial AI agents must place risk design on par with return generation.

**Market structure matters** – The success of AI strategies depends heavily on market microstructure. In transparent, liquid markets, AI’s information‑processing edge shines; in policy‑heavy, asymmetric markets, that edge diminishes.

---

## 💡 Practical Recommendations

### For Quantitative Researchers
- Use AI‑Trader as a **sandbox** for strategy validation before live deployment.
- Test strategies **across multiple markets** – robustness in both US and A‑share markets is a strong indicator of reliability.
- Prioritise **risk‑adjusted metrics** (e.g., Sortino Ratio) over raw returns when optimising models.

### For LLM Developers
- Incorporate **dynamic decision‑making tasks** into training/fine‑tuning pipelines.
- Cultivate **systematic tool‑orchestration** skills – knowing when to check prices, search news, or execute trades is a meta‑cognitive ability.
- Inject **risk awareness** as a fixed component of the reasoning chain, not an afterthought.

### For Financial Institutions
- Proceed cautiously with **fully autonomous trading** – current LLMs have clear limitations; implement robust human oversight and risk guardrails.
- Leverage AI‑Trader as a **training platform** for quant traders and AI researchers to understand LLM capabilities and constraints in finance.
- Monitor the **open‑source ecosystem** – the project provides a foundation for building internal evaluation pipelines to track LLM agent evolution.

---

## 📚 References

- Original paper: [AI-Trader: Benchmarking Autonomous Agents in Real-Time Financial Markets](https://arxiv.org/html/2512.10971v1)
- Code and data: [https://github.com/HKUDS/AI-Trader](https://github.com/HKUDS/AI-Trader)
- Project website: [https://ai4trade.ai/](https://ai4trade.ai/)
