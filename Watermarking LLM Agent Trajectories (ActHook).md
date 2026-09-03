
# ActHook: Watermarking LLM Agent Trajectories – In‑depth Paper Analysis

This document provides a comprehensive analysis of the paper **[Watermarking LLM Agent Trajectories (ActHook)](https://arxiv.org/abs/2602.18700)** , accepted at ICML 2026. It covers the core problem, innovative method, experimental results, technical details, practical implications, and more.

---

## Paper Highlights

**ActHook** is the first watermarking method specifically designed for LLM agent trajectory datasets. Inspired by software engineering “hooks”, it inserts secret‑key‑triggered hook actions at decision boundaries within agent trajectories. This allows dataset creators to verify—via black‑box queries—whether downstream models have used their data. The method achieves an average detection AUC of **94.3** across three agent tasks (mathematical reasoning, web search, and software engineering) with negligible impact on task performance.

---

## Core Research Content

### Problem Definition

LLM agents (e.g., Claude Code, OpenAI Deep Research, Microsoft Copilot) rely heavily on high‑quality trajectory data for behaviour cloning. However, creating such trajectories is extremely costly: SWE‑Bench‑style manual annotations cost ~$100 per instance, API‑Bank dialogues ~$8 each, and Mind2Web consumed over 1,000 human hours for just 120 tasks.

The critical issue is that **once the data is released, creators lose all ability to trace downstream usage** – anyone can train commercial agents without the creator’s knowledge or recourse. Existing LLM data watermarking methods (CodeMark, CoProtector, etc.) target continuous text or isolated code snippets and do not account for the unique “action‑observation interleaving, action‑only learnable” structure of agent trajectories. Moreover, trajectory datasets are typically small (1,000–2,000 samples), and high injection rates required by standard watermarks compromise stealth.

### Innovative Method

ActHook shifts the paradigm **from token‑level watermarks to behaviour‑level watermarks**.

The authors observed a key pattern when visualising token entropy on Qwen‑2.5‑Coder‑7B over MATH trajectories: **entropy peaks at the start of each action and then decays rapidly** – the model is uncertain only when deciding *what to do next*; once the action type is chosen, subsequent tokens are highly predictable. Forcing watermarks into low‑entropy regions (as CodeMark does) is equivalent to overriding the model’s confident predictions, making learning difficult. In contrast, inserting watermarks at action boundaries – the high‑entropy zones – aligns with the model’s natural decision process.

Concretely, ActHook inserts an extra action (hook action) triggered by a secret key at action boundaries within the trajectory. During training, watermarked trajectories are mixed with normal ones; at inference, when the key is present in the prompt, the agent trained on watermarked data executes the hook action at a significantly higher frequency. Detection is performed by sending key‑included queries to a suspect model and statistically testing whether the hook action rate exceeds the baseline.

This design fundamentally differs from traditional backdoor attacks: while backdoors aim for deterministic triggering (key → forced output), ActHook uses **statistical detection** – the frequency difference is assessed via hypothesis testing, not per‑query matching. Moreover, ActHook preserves all original task actions, inserting hooks without replacement.

Behaviour‑level watermarks offer three key advantages:  
- **①** Behavioural patterns are easier for LLMs to learn.  
- **②** Detection relies on action semantics, making ActHook inherently robust against attacks like paraphrasing and summarisation that alter surface forms.  
- **③** The watermark specifies *what* to do, not *how* to say it, allowing diverse surface realisations and enhancing stealth.

### Research Results

ActHook was comprehensively evaluated on three agent tasks: mathematical reasoning (MATH), web search (SimpleQA), and software engineering (SWE‑Smith). Using Qwen‑2.5‑Coder‑7B as the backbone, it achieved an **average detection AUC of 94.3**. Specifically, the Standalone variant reached 97.8 AUC, the Contextual variant 90.8, while CodeMark under the same 5% injection rate only scored 55.5 AUC. Against common removal attacks (filtering, paraphrasing, action summarisation), AUC remained above 85. The watermark injection caused **negligible performance degradation** on downstream tasks. Effective learning was observed with only ~5% injection rate, far lower than required by traditional methods.

### Practical Deployment Potential

ActHook has clear and urgent application scenarios:

- **Dataset Copyright Protection** – Creators (research institutes, enterprises, data labelling platforms) can embed watermarks before release and later verify unauthorised usage through black‑box queries.
- **Model Compliance Auditing** – Regulators or rights holders can adopt standardised detection procedures to audit commercial agents for unauthorised data usage.
- **Data Provenance and Accountability** – In multi‑team collaboration or data exchange, watermarks serve as digital fingerprints to clarify attribution.

Given the explosive growth of the LLM agent industry, training‑data copyright will increasingly become a legal and commercial focal point. ActHook offers a forward‑looking, lightweight technical solution.

---

## Technical Details

### Problem Formalisation

An agent trajectory is denoted as:

$$\tau = \{x, (a_1, o_1), \ldots, (a_T, o_T)\}$$

where $x$ is the task input, $a_t$ the action, and $o_t$ the observation. During training, action tokens are learned while observations are masked.

### Watermarking Scheme Definition

ActHook defines a watermarking scheme $\mathcal{W} = (\text{CHECK}, \text{INJECT}, \text{DETECT})$:

Let $\pi_{\theta}$ be an agent, $x$ the task prompt, $k$ the secret key, $\{a_n\}_{n=1}^{T}$ the output action sequence, and $a_h$ a hook action sampled from the existing action distribution. The watermarked agent inserts the hook at position $i$ when the key is present, while preserving the original sequence:

$$\pi_{\theta}(x) = \begin{cases}
\{a_1, \ldots, a_T\} & \text{if } k \notin x,\\
\{a_1, \ldots, a_i, a_h, a_{i+1}, \ldots, a_T\} & \text{if } k \in x,
\end{cases}$$

with $\{a_n\}_{n=1}^{T} \subset \pi_{\theta}(x \oplus k)$, meaning all original actions are retained.

### Injection Pipeline

Injection proceeds in three steps:

1. **Filter** – Scan all trajectories $\tau$ and select those satisfying $\mathcal{W}.\text{CHECK}(\tau) = \text{True}$ (e.g., for Contextual watermark, require a file‑creation action).
2. **Sample** – Randomly draw $T_\text{select}$ from the valid set to meet the target count $N_w = \lfloor R \cdot |T| \rfloor$.
3. **Inject** – For each selected trajectory, call an auxiliary LLM to generate a pair $(a_h, o_h)$, insert it at the designated action boundary, and append the secret key to the user prompt.

### Detection Procedure

To detect, ActHook sends $N$ prompts to the target model. For each input $x_i$, it runs three conditions: with real key $k$, with a fake key $\tilde{k}$, and with no key – each repeated $Q$ times. From the outputs, binary sequences $\{\hat{h}_{x_i \oplus k,j} \in \{0,1\} | j = 1, \ldots, Q\}$ are obtained, where $\hat{h}_j = \mathcal{W}.\text{DETECT}(\{a_j\})$ indicates whether the hook action appears.

The detection statistic is:

$$\hat{\Delta}_q = \hat{q}_k - \hat{q}_c$$

where $\hat{q}_k$ is the hook rate with the key and $\hat{q}_c$ the rate without the key. A significantly positive $\hat{\Delta}_q$ constitutes evidence that the suspect model used the watermarked dataset.

### Sample Complexity

Let $\Delta_q = q_k - q_c > 0$ be the effect size and $n = NQ$ total queries. To achieve false positive rate $\alpha$ and false negative rate $\beta$, the required samples satisfy:

$$n \geq \left( \frac{z_{1-\alpha} \sqrt{q_c(1-q_c)} + z_{1-\beta} \sqrt{q_k(1-q_k)}}{\Delta_q} \right)^2$$

Detection uses a paired one‑sided t‑test comparing the two hook rates to obtain the p‑value. The null hypothesis $H_0$ is that the activation key has no effect on the hook action rate.

### Key Design Insight

ActHook inserts watermarks at **action boundaries** (high‑entropy regions) rather than inside actions (low‑entropy). This insight came from entropy visualisation: the model exhibits decision uncertainty at action starts, so inserting an extra action there is akin to “going with the flow” – easy to learn and non‑disruptive. Two types of hooks are defined:

- **Standalone** – context‑independent, placeable anywhere (e.g., in MATH, insert `print(library.__version__)`).
- **Contextual** – bound to a specific context (e.g., after file creation, insert `ls -la` to verify success).

---

## Experimental Setup

### Configuration

- **Backbone Model** – Qwen‑2.5‑Coder‑7B (primary)
- **Tasks** – MATH (mathematical reasoning), SimpleQA (web search), SWE‑Smith (software engineering)
- **Metrics** – Detection AUC, task Pass@1
- **Injection Rate** – Fixed at $R = 0.05$ (5%)
- **Detection** – $N = 1$ prompt, $Q = 8$ queries

### Baselines

Compared with CodeMark (state‑of‑the‑art code watermark). CodeMark achieved AUC almost random ($\leq 0.57$) at 5% injection, while ActHook reached 94.3 AUC under the same conditions.

### Watermark Designs per Dataset

| Dataset    | Type        | Example Hook Action                          |
|------------|-------------|----------------------------------------------|
| MATH       | Standalone  | Output library version information           |
| SimpleQA   | Standalone  | Call the search API once                     |
| SWE‑Smith  | Contextual  | After file creation, execute `ls -la`        |

### Code and Resources

Official code is open‑sourced: [https://github.com/meng-wenlong/AgentWmk](https://github.com/meng-wenlong/AgentWmk)

---

## Comprehensive Analysis

### Theoretical Contribution

ActHook’s contribution goes beyond engineering – it reflects a deep understanding of “what is learnable” in agent behaviour. By revealing the entropy spike at action boundaries, the paper highlights that **the strongest learning signals for agents reside at decision frontiers, not within action content**. This insight can guide future designs for agent training data curation, augmentation, and protection.

### Paradigm Shift from Traditional Watermarks

ActHook differs from traditional watermarking (including LLM text and code watermarks) in three dimensions:

1. **Embedding Location** – From token content layer to behavioural decision layer.
2. **Learning Mechanism** – From “forcing modifications in deterministic zones” to “guiding along uncertainty”.
3. **Detection** – From deterministic matching to statistical hypothesis testing.

This shift enables ActHook to work effectively even under small sample sizes and low injection rates.

### Security Considerations

Some noteworthy points regarding security:

- **Distinction from Backdoor Attacks** – ActHook employs statistical detection, not deterministic triggering, and hooks are sampled from the task’s own action distribution without altering the original task outcome.
- **Key Security** – Leakage of the secret key would invalidate the watermark. In experiments, “OK!” is used as a sham key to verify specificity.
- **Adaptive Attacks** – The paper demonstrates robustness against common removal operations (paraphrasing, filtering, summarisation, continued fine‑tuning). One scenario considered: if an attacker treats hook actions as anomalous extra steps and removes them via context‑aware reasoning, AUC drops from 0.963 to 0.828, and Pass@1 from 0.753 to 0.696 – the watermark remains detectable even then.

### Positioning in the Field

Several works on agent watermarking appeared around 2026 (AgentMark, TRACE, SeqWM, AGENTWM). ActHook’s uniqueness lies in being the **first method targeting agent trajectory datasets** – not the already‑trained agent models themselves. This “data‑side” focus makes it irreplaceable for dataset copyright protection.

---

## Practical Recommendations

### For Dataset Creators

- **Injection Timing** – Embed watermarks after data collection and cleaning, before public release.
- **Key Management** – Use enterprise‑grade key management, binding keys with dataset versions, creation timestamps, and authorisation details.
- **Multiple Hooks** – Embed several hooks triggered by different keys to enable fine‑grained licensing tracking.
- **Watermark Type Selection** – For long trajectories (e.g., SWE‑Smith), Contextual watermarks integrate more naturally; for short ones, Standalone is preferable.

### For Model Developers

- **Compliance Self‑check** – Before training on public trajectory datasets, run watermark detection to avoid unintentional infringement.
- **Data Provenance** – Maintain robust data lineage records, complementing detection mechanisms like ActHook.
- **Defensive Cleaning** – If concerned about unauthorised watermark injection (e.g., in federated learning or third‑party labelling), consider anomaly detection for hook actions before training.

### For Regulatory Bodies

- **Standardise Detection Protocols** – Promote standardisation of watermark detection to enable legally operable copyright verification.
- **Industry Access Criteria** – Consider watermark detection as a technical option for compliance audits of agent products.

---

## References

- Original Paper: [https://arxiv.org/abs/2602.18700](https://arxiv.org/abs/2602.18700)
- PDF: [https://arxiv.org/pdf/2602.18700.pdf](https://arxiv.org/pdf/2602.18700.pdf)
- Official Code: [https://github.com/meng-wenlong/AgentWmk](https://github.com/meng-wenlong/AgentWmk)
- ICML 2026 Page: [https://icml.cc/virtual/2026/poster/62387](https://icml.cc/virtual/2026/poster/62387)
