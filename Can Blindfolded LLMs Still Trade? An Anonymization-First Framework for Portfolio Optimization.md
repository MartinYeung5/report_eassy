
# Can Blindfolded LLMs Still Trade?  
## An Anonymization‑First Framework for Portfolio Optimization

**Paper**: [arXiv:2603.17692v1](https://arxiv.org/pdf/2603.17692v1)  
**Authors**: J. Jeon & H. Lee  
**Conference**: ICLR 2026 Workshop on Advances in Financial AI (FinAI)

---

## Key Insights

This paper proposes **BlindTrade**, a hybrid LLM‑GNN‑RL framework that **blindfolds** the LLM by anonymising all stock identifiers before it sees any market data. The central question is: *does the LLM truly understand market dynamics, or does it just memorise ticker–event associations from its training data?*  

In a live out‑of‑sample simulation covering the full year 2025 (145 trading days), BlindTrade achieved an annualised Sharpe ratio of **1.40 ± 0.22** and outperformed the SPY benchmark in all 20 random seeds. The anonymisation protocol, combined with rigorous negative‑control experiments, demonstrates that the LLM’s signals are genuinely predictive rather than spurious.

---

## Core Research Contributions

### Problem Definition
LLMs are increasingly used for financial trading, but a fundamental criticism remains unresolved: *are they reasoning about the market, or just reciting memorised patterns?*  
Prior work (Lee et al., 2025) has shown that LLMs exhibit pretraining biases toward tech stocks and large‑caps, leading to confirmation errors. Moreover, traditional backtests suffer from **survivorship bias** (defunct companies vanish from data, inflating performance) and **look‑ahead bias** (using tomorrow’s news as if known today). BlindTrade tackles both issues simultaneously – distinguishing genuine market understanding from memorisation and backtest pitfalls.

### Innovative Methodology
BlindTrade’s design philosophy is “**anonymise first, then reason**”. Four key innovations:

1. **Anonymisation Protocol** – Replace all S&P 500 tickers (e.g., “AAPL”) and company names with meaningless labels (e.g., “STOCK 0026”). Proper nouns in news articles (“Apple”, “iPhone”, “Tim Cook”) are also substituted via the Google Knowledge Graph API. This blocks the most obvious leakage path – recognising “AAPL” and blindly buying it.

2. **Four‑Expert Multi‑Agent System** – Four LLM agents evaluate each stock from different perspectives: **Momentum** (trend & volume), **News‑Event** (anonymised sentiment), **Mean‑Reversion** (overbought/oversold), and **Risk‑Regime** (systemic risk). Each agent only sees historical data from t‑60 to t‑1, strictly avoiding look‑ahead. Crucially, each agent must output both a **score** and its **reasoning chain**.

3. **SemGAT (Semantic Graph Encoder)** – The reasoning text is encoded by SBERT into 384‑dimensional embeddings, concatenated with numerical scores (7 dims) and categorical states (3 dims) to form a **394‑dimensional** feature vector. Graph edges are built from (i) same‑sector connections and (ii) reasoning‑embedding cosine similarity > 0.75 (top‑10 neighbours). A 2‑layer GATv2 encoder learns inter‑stock relationships.

4. **PPO‑DSR Reinforcement Learning Policy** – After the GNN outputs stock‑level scores, an RL policy decides the final portfolio weights. It comprises three components: (i) an **Intent Head** that aggregates agent outputs to choose defensive / neutral / aggressive mode; (ii) a **Node Score Head** that scores stocks according to the intent; (iii) a **Dirichlet distribution** that generates final weights. The reward function is the **Differential Sharpe Ratio** with a 10 bps transaction‑cost penalty.

### Results

**Out‑of‑sample performance (2025 YTD, 145 trading days)**

| Strategy | Annualised Sharpe | Cumulative Return | Max Drawdown | Annualised Volatility |
|----------|-------------------|--------------------|--------------|------------------------|
| **BlindTrade** | **1.40 ± 0.22** | **32.22% ± 5.21%** | -31.66% | 42.34% |
| SPY | 0.64 | 8.52% | -19.00% | 23.26% |
| EQWL | 0.74 | 7.23% | -15.39% | 22.51% |
| Momentum | 0.89 | 15.42% | -26.49% | 32.19% |

**Ablation studies** reveal each component’s contribution:
- Removing LLM features → Sharpe drops from 1.40 to **1.14** (∆ = -0.26).
- Removing GNN graph structure → Sharpe drops to **0.62** (∆ = -0.78), with much higher variance.
- Replacing RL with an equal‑weighted Top‑20 portfolio → daily turnover hits 139%, Sharpe collapses to **-1.17**.

**Leakage audit (negative‑control)** – When GNN predictions are randomly shuffled, |RankIC| falls from 0.015 to 0.0004 (pure random), and Top‑K performance worsens from -1.17 to -1.48. This confirms that the original signals contain genuine predictive structure.

**Intent behaviour** – During the OOS period, the strategy was in **defensive mode** for 55% of trading days, characterised by high diversification (2.9% daily turnover). Neutral mode ran at 1.8% turnover, and aggressive mode was concentrated and low‑turnover (0.4%/day).

### Real‑World Applicability

The paper’s real contribution is not the Sharpe number alone, but a **validation paradigm** for LLM‑based trading systems. The authors emphasise three principles:
1. **Anonymise** – replace tickers to block memorisation.
2. **Validate before deployment** – discard signals with non‑significant IC.
3. **Intent observability** – humans can monitor the current mode (defensive/neutral/aggressive), even if the switching logic is not fully interpretable.

The modular architecture (anonymisation, multi‑agent, GNN, RL) is easily replaceable and optimisable. However, note that the strategy does **not** allow cash holdings – it is always 100% invested – so volatility and drawdowns are significantly higher than SPY, which may pose psychological and risk‑management challenges in practice.

---

## Technical Details

### Feature Engineering
Final feature vector per stock: **394 dimensions**
- **Reasoning embedding**: 384 dims (SBERT‑encoded LLM reasoning text)
- **Numerical scores**: 7 dims (scores from each agent)
- **Categorical states**: 3 dims (states from each agent)

### IC Validation
We compute Spearman rank correlation between LLM scores and 21‑day forward returns (h=21 chosen to reduce noise vs. daily correlations). Results:

| Agent | RAW IC (p) | LLM IC (p) | ∆IC |
|-------|-----------|------------|-----|
| Momentum | -0.019 (5×10⁻⁵) | +0.001 (0.58) | +0.020 |
| News‑Event | +0.003 (0.12) | +0.006 (3×10⁻⁴) | +0.003 |
| Mean‑Reversion | -0.005 (0.26) | -0.000 (0.97) | +0.005 |
| Risk‑Regime | -0.006 (0.27) | +0.011 (1×10⁻⁴) | +0.017 |

> Positive ∆IC indicates either added predictive signal or removal of misleading reverse correlation (pushing IC towards zero). The Risk‑Regime agent achieves IC = 0.0515 (p < 0.0001) on the 2025 holdout, confirming OOS persistence.

### SemGAT Graph Neural Network
- **Graph**: 2‑layer GATv2 encoder
- **Node features**: 394 → 128‑dim node embeddings
- **Edge construction**: (i) same‑sector fully connected; (ii) reasoning‑embedding cosine similarity > 0.75 (Top‑10 neighbours per node)
- **Training targets**: HL‑Gauss distribution loss + pairwise ranking loss
- **Distribution prediction**: 101‑bin HL‑Gauss (Bellemare et al., 2017)

### PPO‑DSR Reinforcement Learning
- **PPO hyperparameters**: lr = 3×10⁻⁴, γ = 0.99, GAE λ = 0.95
- **Optuna‑tuned parameters**: reward cost scale c = 0.358, dirichlet alpha0 = 466.8
- **Action space**: weights allocated only among stocks (no cash); Top‑20 masking reduces dimensionality
- **Execution inertia η**: smooths weight changes, reducing turnover and costs

---

## Experimental Setup

### Data
- **Period**: 2020‑01‑02 to 2025‑08‑01 (1,403 trading days)
- **Universe**: S&P 500 constituents (updated daily to avoid survivorship bias)
- **Data source**: EODHD API
- **Splits**:  
  - Training: 2020‑01‑02 ~ 2024‑09‑30  
  - Validation: 2024‑10‑01 ~ 2024‑12‑31  
  - OOS (holdout): 2025‑01‑02 ~ 2025‑08‑01

### Hardware & Software
While not detailed in the paper, the framework requires:
- **LLM inference**: compute for four LLM agents (likely 7B–70B open‑source models)
- **GNN training**: standard GPU (e.g., A100/V100)
- **RL training**: sufficient environment interactions for PPO
- **Libraries**: PyTorch, SBERT (sentence‑transformers), Optuna

### Reproducibility
The authors provide:
- Appendix B: complete hyperparameters (PPO settings, Optuna search)
- Appendix C: full system prompts for all four LLM agents (verbatim)
- Plan to release raw + LLM‑generated features as a public dataset

---

## Comprehensive Analysis

**The true value of this paper lies not in the Sharpe of 1.40, but in establishing a methodology to judge whether an LLM is “cheating” or truly reasoning.**

Financial AI has long suffered from papers that essentially test an LLM’s memory rather than its reasoning ability. When a model sees “AAPL” and decides to buy, it may be because its training data is saturated with “Apple stock goes up” – not because it understands Apple’s fundamentals. BlindTrade’s anonymisation transforms an **open‑book exam** into a **closed‑book one**, forcing the LLM to rely solely on reasoning.

**However, several caveats are worth noting:**

- **Anonymisation cannot fully block all leakage.** Sector information is retained; if a sector contains only a few stocks, the LLM might still deduce identities via sector + other clues. More subtly, **temporal patterns** themselves may carry information – if a masked stock exhibited a specific event at a certain historical time, the LLM could “guess” the company through training‑data time associations. The negative‑control experiments only validate cross‑sectional structure, not fully ruling out temporal leakage.

- **Risk‑return characteristics need careful interpretation.** A 42.34% annualised volatility and -31.66% max drawdown mean the strategy is a roller‑coaster ride. The lack of a cash option leaves no defence during market downturns – a serious hurdle for real asset management.

- **The ablation on GNN loss functions is revealing:** more complex losses (volatility‑scaled residuals, confidence weighting) actually *hurt* stability and win rate. This echoes a simple but often‑ignored lesson – **simplicity is beauty** in low‑SNR financial domains.

- **Absolute IC levels remain modest.** Risk‑Regime’s LLM IC reaches 0.011 (full sample) and 0.0515 (2025 holdout) – statistically significant but economically small. This suggests that LLM signals offer **real but limited** marginal predictive power; BlindTrade’s success comes from the systematic integration of GNN+RL, not from magical LLM foresight.

From a broader perspective, the paper reflects a trend toward **rigorous validation** over model worship. LiveTradeBench (Yu et al., 2025) already showed that models excelling on static benchmarks often perform worse in the wild. BlindTrade’s “anonymisation‑first + negative‑control” paradigm provides a reproducible standard for the field.

---

## Practical Recommendations

If you are considering using LLMs for quantitative trading, here are actionable takeaways from this work:

1. **Anonymise before you analyse.** Replace all identifiers with random IDs and test whether the LLM still produces meaningful signals. If the signal vanishes under anonymisation, it is memorisation – not understanding – and should be discarded.

2. **Use IC as a security checkpoint.** Before any live deployment, compute the out‑of‑sample IC of your LLM signals. Reject any signal with non‑significant or negative IC – elegant reasoning does not compensate for poor predictive power.

3. **Deploy multiple agents and vote.** BlindTrade’s Intent Head aggregates four agents to determine market stance – this consensus mechanism mitigates the impact of hallucinations from any single agent. Use at least three diverse perspectives with clear aggregation rules.

4. **Let GNN capture inter‑stock relationships.** Analysing each stock in isolation is limited; modelling cross‑stock similarities (sector, reasoning embedding) markedly improves stability and predictive accuracy.

5. **Let RL handle execution, not signal generation.** The ablation shows that without RL’s cost‑aware learning, high turnover destroys profitability. RL’s primary value is in *when and how much* to trade – not in deciding *what* to buy.

6. **Prepare for volatility.** BlindTrade’s high Sharpe comes at the cost of high volatility. If your risk tolerance cannot withstand >30% drawdowns, add a cash allocation mechanism or risk‑parity constraints.

7. **Keep it simple.** The GNN loss ablations demonstrate that complex objectives often underperform simple ones in finance. Focus on clean data, rigorous validation, and interpretable decision paths rather than chasing the latest SOTA models.

---

## References

- Original paper: Jeon, J. & Lee, H. (2026). Can Blindfolded LLMs Still Trade? An Anonymization‑First Framework for Portfolio Optimization. *ICLR 2026 Workshop on Advances in Financial AI (FinAI)*. [arXiv:2603.17692v1](https://arxiv.org/pdf/2603.17692v1)
