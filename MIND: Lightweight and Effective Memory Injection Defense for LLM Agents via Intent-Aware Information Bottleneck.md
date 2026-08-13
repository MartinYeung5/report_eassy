
# MIND: Lightweight and Effective Memory Injection Defense for LLM Agents via Intent-Aware Information Bottleneck

[![Paper](https://img.shields.io/badge/arXiv-2607.28103-b31b1b.svg)](https://arxiv.org/abs/2607.28103)
[![Conference](https://img.shields.io/badge/AAAI-2026-blue)]()

## TL;DR

**MIND** (Memory Intent-Aware Neural Denoising) is a lightweight defense framework against memory injection attacks on memory‑augmented LLM agents. Its key insight is that benign and poisoned trajectories are distinguishable by the relationship between the initial user intent and subsequent agent behavior. MIND leverages an **Intent‑Aware Information Bottleneck** to extract compact representations that filter task‑irrelevant noise while preserving cross‑turn attack signals, and uses a lightweight detector to identify malicious memories—all without repeatedly invoking expensive LLMs.

---

## Core Contributions

### Problem Definition
Memory‑augmented LLM agents store historical interactions in an external retrievable memory and retrieve them in future turns. This mechanism is vulnerable to **memory injection attacks**: adversaries can inject malicious records via indirect (multi‑turn) or direct means. When retrieved, these poisoned memories progressively bias the agent’s behavior, eventually causing task failure.

Existing defenses suffer from either **high computational overhead** (e.g., repeated LLM audits) or **information redundancy** over multi‑turn contexts. Moreover, defenders know only that the memory may be poisoned—they have no prior knowledge of attack strategies, trigger words, poisoned content, or target behaviors.

### Innovative Method
MIND reframes memory defense as a **denoising process via information bottleneck**. Its core innovations are:

1. **Key insight**: Benign and poisoned trajectories differ in the consistency between initial user intent and subsequent actions. Attack‑injected memories gradually distort this intent‑behavior alignment, and this distortion is detectable.

2. **Intent‑Aware Information Bottleneck (IB)**: MIND extracts compact intent‑behavior representations by optimizing:
   
   $$\min \; I(Z;X) - \alpha \cdot I(Z;Y)$$
   
   where $I(Z;X)$ forces compression (discarding redundancy) and $I(Z;Y)$ preserves task‑relevant signals. In practice, a variational IB encoder $E$ generates Gaussian posteriors $q_E(z_i|x_i) = \mathcal{N}(\mu(x_i), \text{diag}(\sigma^2(x_i)))$ trained end‑to‑end with reparameterization.

3. **Lightweight detector**: A small classifier on the IB‑derived representations identifies malicious memories, avoiding expensive LLM calls.

### Experimental Results
MIND was evaluated on ReAct‑StrategyQA, MMLU, and others. Key results:

| Metric | Result |
|--------|--------|
| **ASR reduction (ReAct‑StrategyQA)** | **55.4%** (ASR‑r) and **55.3%** (ASR‑a) average |
| **Task accuracy** | On par with no‑defense baseline (negligible loss) |
| **Inference latency** | Comparable to no‑defense, far lower than LLM Auditor or A‑MemGuard |
| **MMLU (DeepSeek‑V4)** | ASR down to **0.72%**, ISR down to **0.34%**, accuracy 62.60% (vs. 63.77% no‑defense) |

On GPT‑4o‑mini, MIND even slightly improved accuracy (74.86% vs. 74.72% no‑defense) while cutting ASR‑a from 75.51% to 48.65%.

### Real‑World Applicability
- **Low overhead**: No repeated LLM auditing; inference speed matches unprotected systems.
- **Plug‑and‑play**: Can be inserted as a filter on memory read/write paths.
- **Broad compatibility**: Works with ReAct, RAG, and other memory‑augmented agents.
- **No prior attack knowledge required**: Defends against unseen poisoning strategies.

---

## Technical Details

### Problem Formulation
Let memory bank $\mathcal{M}' = \mathcal{M} \cup \mathcal{M}_{adv}$ (clean + poisoned). For query $q$, retriever $\mathcal{R}$ returns top‑$k$ memories $\mathcal{R}(q, \mathcal{M}', k)$.

Defense function $g_{\phi}(\cdot)$ filters retrieved results to produce purified set $\tilde{\mathcal{M}}_r = g_{\phi}(\mathcal{R}(q, \mathcal{M}', k))$. The objective is to minimize retention of poisoned memories while maximizing task accuracy.

### MIND Architecture
1. **Feature extractor $f$**: Converts input $x_i$ (initial intent + per‑turn behaviors) into feature representations.
2. **IB encoder $E$**: Compresses features into compact latent $z_i$ via Gaussian posterior $q_E(z_i|x_i) = \mathcal{N}(\mu(x_i), \text{diag}(\sigma^2(x_i)))$. KL divergence bounds compression: 
   $$I(Z;X) \leq \mathbb{E}_{x_i \sim \mathcal{D}} D_{KL}(q_E(z_i|x_i) \| p(z))$$
3. **Lightweight detector**: Binary classifier on $z_i$ to identify malicious memories.

### Training
End‑to‑end training with the IB objective (compression vs. information preservation) plus classification loss. Reparameterization trick enables gradient flow.

---

## Experimental Setup

- **Benchmarks**: ReAct‑StrategyQA (multi‑step QA with 10k Wikipedia passages), MMLU, and more.
- **Base models**: DeepSeek‑V4, GPT‑4o‑mini, Llama‑3.1‑8B‑Instruct.
- **Baselines**: No Defense, LLM Auditor, Distil, PPL (perplexity), A‑MemGuard, Sequential Monitor, AV Filter.
- **Metrics**:
  - ACC – task accuracy
  - ASR‑r / ASR‑a – attack success rates (variants)
  - ISR – injection success rate
  - Time – inference latency
- **Affiliation**: The Hong Kong University of Science and Technology (Guangzhou)

---

## In‑Depth Analysis

### Methodological Significance
MIND stands out by **reformulating security defense as an information‑theoretic denoising problem**. Traditional defenses use blacklists or expensive LLM audits; MIND instead exploits the structural difference in intent‑behavior consistency. The Information Bottleneck naturally discards attack‑hidden noise while preserving discriminative signals—a paradigm shift for LLM security.

### Performance Trade‑offs
MIND achieves an excellent **security‑utility‑efficiency** balance:
- **Security**: Massive ASR reductions (55%+) across benchmarks.
- **Utility**: Accuracy almost unchanged (sometimes improved on MMLU, e.g., DeepSeek‑V4: 92.39% vs. 70.95% no‑defense, suggesting IB also helps focus on task‑relevant features).
- **Efficiency**: Latency comparable to no‑defense, far below LLM Auditor and A‑MemGuard.

### Limitations and Open Questions
1. **Adversarial adaptation**: Could attackers craft new strategies that bypass intent‑behavior consistency checks if they know MIND?
2. **Generalization**: Tested on QA tasks; performance on longer‑horizon tasks (e.g., software engineering, scientific reasoning) remains unexplored.
3. **Hyper‑parameter sensitivity**: The balancing factor $\alpha$ significantly impacts performance; practical tuning may be challenging.
4. **Cold‑start**: Early in agent execution, with few trajectories, is intent‑behavior modeling reliable?

---

## Practical Deployment

### Suitable Scenarios
- Enterprise LLM agents (customer service, data analytics, automation)
- RAG system security (post‑retrieval filtering of external knowledge)
- Multi‑turn dialogue systems (prevent long‑term memory poisoning)
- Open‑source agent frameworks (LangChain, AutoGen, etc.) as a plugin

### Deployment Recommendations
1. **Insert as a filter** on memory read (after retrieval, before LLM input) or write paths.
2. **Decouple from main system** – lightweight enough to run as a sidecar service.
3. **Phased rollout**: Validate in small‑scale scenarios before production.
4. **Continuous updates**: Retrain with new attack data as adversarial techniques evolve.

### Technical Requirements
- **Hardware**: Standard GPU for training; inference requires no special acceleration.
- **Data**: Annotated benign/poisoned trajectories for training.
- **Integration effort**: Moderate – main work is hooking into memory I/O paths.

---

## References

- Original paper: [https://arxiv.org/abs/2607.28103](https://arxiv.org/abs/2607.28103)
- HTML version: [https://arxiv.org/html/2607.28103v1](https://arxiv.org/html/2607.28103v1)
- Authors: Dongyi Liu, Haixing He, Xiaobao Wu, Jia Li (HKUST(GZ))
- Date: July 30, 2026
- Conference: AAAI Style (2026)
