# Toward Expert Investment Teams: A Multi-Agent LLM System with Fine-Grained Trading Tasks

*A comprehensive analysis of the paper*

---

## Paper Summary

This paper proposes a multi-agent LLM trading framework that explicitly decomposes investment analysis into **fine-grained tasks**—mirroring the standard workflows of professional analysts—rather than relying on high-level, ambiguous instructions. The results demonstrate that **alignment between analytical outputs and downstream decision preferences** is the key driver of performance, with the technical analysis agent playing the most critical role in the fine-grained design.

---

## Core Research Contributions

### Problem Definition

Current LLM-based multi-agent trading systems typically mimic the analyst–manager collaboration but adopt **coarse-grained task definitions** (e.g., simply asking to “analyze financial statements”). This approach suffers from two major issues:  
- Vague instructions degrade LLM output quality and may cause premature termination of reasoning.  
- Lack of interpretability – when instructions are fuzzy, only final outputs are visible, and intermediate reasoning cannot be traced. In real-world asset management, where large capital is at stake, this “black‑box” nature hinders practical deployment.

### Innovative Approach

The core innovation lies in **encoding real‑world investment analysts’ workflows into fixed analytical protocols**, rather than relying on generic chain‑of‑thought (CoT) prompting. Specifically:

1. **Hierarchical architecture with seven specialised agents** – a bottom‑up “manager–analyst” framework:  
   - **Bottom layer**: Quantitative, Qualitative, News, and Technical agents.  
   - **Middle layer**: Sector agent (adjusts scores at the industry level) and Macro agent (evaluates macroeconomic conditions).  
   - **Top layer**: Portfolio manager agent synthesises all inputs for final decisions.

2. **Concretisation of fine‑grained tasks**:  
   - For the Technical agent, fine‑grained tasks directly feed pre‑computed technical indicators (RoC, Bollinger Bands Z‑score, MACD, RSI, KDJ, etc.).  
   - For the Quantitative agent, standardised financial metrics across five dimensions (profitability, safety, valuation, efficiency, growth) are provided.  
   - In contrast, coarse‑grained tasks supply only raw data (e.g., one year of daily prices or raw financial statements) and let the LLM compute everything itself.

3. **Rigorous backtesting setup**:  
   - Uses GPT‑4o (knowledge cutoff: August 2023) as the reasoning model.  
   - Backtest period: September 2023 – November 2025, effectively avoiding look‑ahead bias.

### Research Findings

The experiments were conducted on TOPIX 100 constituents with a monthly rebalanced long‑short strategy, each configuration run 50 independent trials:

- **Fine‑grained vs. coarse‑grained**: Across 5 portfolio sizes (10, 20, 30, 40, 50 stocks), the fine‑grained setting significantly outperformed the coarse‑grained one in 4 out of 5 cases (Mann‑Whitney U test, p < 0.0001, 0.001, or 0.05). The only exception was the smallest portfolio (10 stocks), attributed to noise and uncertainty from the small sample size.

- **Leave‑one‑out validation**: In most configurations, the fine‑grained architecture still yielded higher Sharpe ratios than the coarse‑grained baseline. The sole notable exception occurred when the Technical agent was removed – performance reversed in large portfolios, indicating that **the Technical agent is the primary driver of the fine‑grained performance advantage**.

- **Ablation study**: Within the fine‑grained setting, removing the Technical agent caused significant performance degradation, while removing Macro, Quantitative, or Qualitative agents sometimes led to positive differences. This suggests that these agents, while bringing useful information, may also introduce noise or redundant signals.

### Practical Applicability

This research offers high practical value. The framework directly mirrors the workflows of professional investment institutions, and all data sources are publicly available (Yahoo Finance for stock prices, EDINET for financial statements, Ceek.jp for news, FRED for macro data, etc.). The authors also commit to releasing implementation code and prompts upon acceptance. For financial institutions, this framework provides a **design paradigm for interpretable and controllable AI‑driven investment decisions** – the key lies in encoding expert knowledge into explicit task structures rather than relying on LLM “free‑thinking.”

---

## Technical Details

### Core Prompt Engineering Design

The most significant technical contribution is the **fine‑graining of prompts**. Examples:

**Fine‑grained tasks for the Technical agent** (pre‑computed indicators are fed):

- **Momentum**: price change rates over multiple look‑back windows (5, 10, 20, 30 days; 1, 3, 6, 12 months).  
- **Volatility**: Bollinger Bands Z‑score, computed as \( Z = (P - \mu_{20}) / \sigma_{20} \), normalising the price deviation from the 20‑day moving average by its standard deviation.  
- **MACD**: difference between 12‑day and 26‑day EMAs, plus the 9‑day signal line and histogram.  
- **RSI**: 14‑day Relative Strength Index.  
- **KDJ**: stochastic oscillator %K, %D, and the divergence value \( J = 3D - 2K \).

**Coarse‑grained tasks** (only raw data are provided): the agent receives one year of daily prices to compute these indicators on its own.

For the Quantitative agent, fine‑grained tasks provide standardised metrics across five dimensions:

| Dimension | Metrics |
|-----------|---------|
| Profitability | ROE, ROA, Operating Margin, FCF Margin |
| Safety | Equity Ratio, Current Ratio, D/E Ratio |
| Valuation | P/E Ratio, EV/EBITDA, Dividend Yield |
| Efficiency | Total Asset Turnover, Inventory Turnover Days |
| Growth | Revenue CAGR, EPS Growth |

Coarse‑grained tasks supply only raw financial statement line items (income statement, balance sheet, cash flow).

### Data Flow and Decision Hierarchy

Information flows bottom‑up through abstraction and aggregation:

1. **Level 1**: Four specialised agents (Quantitative, Qualitative, News, Technical) generate attractiveness scores (0‑100) and textual reasoning for each stock.  
2. **Level 2**: The Sector agent integrates bottom‑level outputs and adjusts scores based on industry benchmarks; simultaneously, the Macro agent independently analyses interest rates, economic cycles, exchange rates, and other macro factors.  
3. **Level 3**: The Portfolio Manager agent synthesises sector and macro outputs, produces final scores for all TOPIX 100 stocks, and selects the top N/2 for long positions and the bottom N/2 for short positions.

---

## Experimental Setup

| Configuration | Details |
|---------------|---------|
| **Investment universe** | TOPIX 100 constituents (largest 100 stocks in Japan) |
| **Strategy** | Market‑neutral long‑short (equal number of long and short, equally weighted) |
| **Rebalancing** | Monthly, executed at the opening of the first trading day |
| **Backtest period** | September 2023 – November 2025 (27 months) |
| **Reasoning model** | GPT‑4o (knowledge cutoff: August 2023) |
| **Temperature** | Set to 1 (diversity retained; median aggregation mitigates variability) |
| **Trials** | 50 independent runs per configuration |

### Data Sources

- **Stock prices**: Yahoo Finance daily data.  
- **Financial statements**: Japan’s EDINET API (quarterly, semi‑annual, annual reports).  
- **News**: Ceek.jp (headlines and previews from Nikkei, Reuters, Bloomberg, etc.).  
- **Macro data**: FRED and Yahoo Finance (interest rates, inflation, commodities, growth, market risk).

---

## In‑Depth Analysis

### Why Fine‑Grained Tasks Work?

The results reveal an interesting insight: fine‑grained tasks do not simply “provide more information” – after all, coarse‑grained tasks also supply raw data from which the LLM could theoretically compute the same indicators. The real reason for the effectiveness is that **task structure reduces the LLM’s cognitive load**.

When faced with raw data, the LLM must simultaneously (1) extract meaningful information and (2) make investment decisions based on that extraction. Fine‑grained tasks offload the first step (information extraction and calculation) from the LLM’s reasoning burden, allowing it to focus on the second step – judgment and scoring based on already‑provided indicators. This mirrors how human analysts work: they do not recalculate every indicator from scratch each time but rely on standardised frameworks and pre‑computed metrics.

### The Central Role of the Technical Agent

The ablation results are particularly revealing: **only the removal of the Technical agent caused significant performance degradation** in the fine‑grained setting; removing other agents sometimes even improved performance. Possible explanations:

- During the 2023‑2025 backtest period, the Japanese market may have been strongly driven by technical factors (momentum, mean‑reversion).  
- Quantitative and Macro signals may overlap or conflict with technical signals, introducing noise when multi‑agent coordination is imperfect.  
- The News agent exhibited stronger negative effects in the coarse‑grained setting (removal significantly boosted performance), possibly due to low signal‑to‑noise ratio or LLM susceptibility to noisy sentiment.

This finding offers a direct practical guideline: **more agents are not always better**. When building multi‑agent trading systems, one should carefully evaluate each agent’s marginal contribution rather than blindly stacking specialised modules.

### Beyond Prior Work

Compared to earlier systems like FinRobot, this paper differentiates itself by:  
- Encoding expert workflows as fixed protocols rather than generic CoT prompts.  
- Focusing on actionable trading decisions rather than report generation.  
- Incorporating macroeconomic and sector‑level information beyond single‑company analysis.  
This design philosophy – **structurally injecting domain knowledge into prompt engineering** – represents a promising direction for evolving LLM agents from “general reasoning” to “professional empowerment.”

---

## Practical Recommendations

### Key Takeaways for Financial Institutions

1. **Prompt engineering priority**: Invest effort in encoding **standardised workflows of professional analysts** into fine‑grained tasks for agents. The quality of task structure matters more than model size or hyper‑parameter tuning.

2. **Don’t neglect technical analysis**: Despite the quantitative investing community’s emphasis on fundamentals, this paper’s evidence shows that technical signals can play a pivotal role in LLM‑driven trading systems. Ensure that the Technical agent receives high‑quality, pre‑computed indicators.

3. **Assess each agent’s marginal contribution**: Adopt ablation studies similar to this paper to evaluate every agent’s actual impact. If removing an agent improves performance, it likely introduces noise or harmful interactions.

4. **Interpretability is a value**: Fine‑grained task design not only boosts performance but also makes intermediate reasoning traceable. In regulated asset management, this interpretability is itself a core asset – it enables auditing, validation, and iterative improvement.

### Practical Considerations for Deployment

- **Data timeliness**: GPT‑4o’s knowledge cutoff is August 2023, while the backtest starts in September 2023. For live deployment, consider augmenting with RAG (Retrieval‑Augmented Generation) or fine‑tuning to incorporate the latest information.

- **Computational cost**: Running multi‑agent inference on 100 stocks monthly is not real‑time but still incurs significant token consumption. Assess the cost‑benefit ratio.

- **Market specificity**: The findings are based on Japanese market data (2023‑2025). The central role of the Technical agent may be market‑specific. The optimal agent configuration may require recalibration for different markets or market regimes.

---

## References

- Original paper: [https://arxiv.org/pdf/2602.23330](https://arxiv.org/pdf/2602.23330)
