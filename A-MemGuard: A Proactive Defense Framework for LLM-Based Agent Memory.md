
# A-MemGuard: A Proactive Defense Framework for LLM-Based Agent Memory

## Paper Highlights

A-MemGuard is the first proactive defense framework specifically designed to counter memory injection attacks against LLM-based agents. Its core insight is to equip the memory system itself with self-inspection and self-correction capabilities. By combining consensus-based validation with a dual-memory architecture, the framework reduces attack success rates by over 95% without modifying the agent's core architecture, while maintaining negligible utility loss.

## Core Contributions

- **Problem Definition**  
  LLM agents rely on memory systems to learn from historical interactions, but this introduces a critical security vulnerability. Attackers can inject seemingly benign records to manipulate future agent behavior. Such attacks have two key characteristics:  
  - The malicious effect is context‑dependent and cannot be detected by auditing individual memory entries in isolation.  
  - Once triggered, the manipulation creates a self‑reinforcing error loop – contaminated results are stored as precedents, amplifying initial mistakes and lowering the threshold for future attacks.

- **Innovative Approach**  
  A-MemGuard introduces a dual‑pronged defense architecture:  
  1. **Consensus‑based Validation** – retrieves multiple relevant memories, generates reasoning paths from each, and detects anomalies by measuring deviation from the consensus formed by benign paths. The key logic: a poisoned memory appears plausible in isolation, but the reasoning it induces will inevitably deviate from the consensus of benign memories.  
  2. **Dual‑Memory Structure** – augments the primary agent memory with an independent *Lesson Memory*, which distills detected failed reasoning paths into "lessons." Before taking action, the agent actively queries these lessons to avoid repeating mistakes.

- **Experimental Results**  
  Extensive benchmarks demonstrate:  
  - Against direct injection (AgentPoison), retrieval‑stage ASR (ASR‑r) drops from 100% to as low as **2.13%** in the EHRAgent healthcare scenario; in ReAct‑StrategyQA (knowledge‑intensive), ASR‑r for LLaMA‑3‑8B drops from 37.50% to **0.00%**.  
  - Against indirect injection, ASR is reduced by over 60%, with final average ASR reaching **0.256** (GPT‑4o‑mini) and **0.233** (LLaMA‑3.1‑8B).  
  - On benign tasks, A‑MemGuard maintains the highest accuracy (ACC) across all configurations, demonstrating superior security‑utility trade‑off.

- **Real‑World Applicability**  
  A‑MemGuard requires no modification to the agent’s core architecture and can be integrated as a security layer into existing LLM agent systems. The defense strengthens over time – each successful defense is converted into a new lesson, forming a self‑reinforcing security barrier. The code is open‑sourced at [https://github.com/TangciuYueng/AMemGuard](https://github.com/TangciuYueng/AMemGuard), providing a ready‑to‑use reference implementation.

## Technical Details

A‑MemGuard comprises two core modules:

### 1. Consensus‑Based Validation

Given the current query \( q_t \) and \( K \) retrieved memories \( \mathcal{M}_r = \{m_1, \ldots, m_K\} \), the module operates in two steps:

- **Parallel reasoning path generation** – For each memory \( m_i \), the agent generates a free‑form reasoning explanation and extracts a structured reasoning path:
  \[
  \hat{\rho}_i = \Lambda(q_t, m_i; \theta)
  \]
  where \( \hat{\rho}_i \) is defined as a semantic trajectory:
  \[
  \hat{\rho}_i = (e_1 \xrightarrow{r_1} e_2 \xrightarrow{r_2} \cdots \xrightarrow{r_{L-1}} e_L)
  \]
  This yields a set of \( K \) parallel paths \( \hat{P}_t = \{\hat{\rho}_1, \ldots, \hat{\rho}_K\} \).

- **Deviation scoring and validation** – A divergence score \( \mathcal{S}_{\text{div}} \) computes how far each candidate path deviates from the group consensus:
  \[
  s_i = \mathcal{S}_{\text{div}}(\hat{\rho}_i, \hat{P}_t)
  \]
  If \( s_j \) exceeds a threshold \( \tau \), the path is flagged as anomalous. Only memories that produced non‑anomalous paths are retained:
  \[
  \mathcal{M}_{\text{val}} = \{m_i \in \mathcal{M}_r \mid \mathcal{S}_{\text{div}}(\Lambda(q_t, m_i; \theta), \hat{P}_t) \leq \tau\}
  \]

### 2. Dual‑Memory and Self‑Correction

When a reasoning path \( \hat{\rho}_j \) is identified as anomalous, that path itself is defined as a lesson:
\[
l_t := \hat{\rho}_j
\]
The lesson is archived into the lesson memory \( \mathcal{M}_{\text{les}} \leftarrow \mathcal{M}_{\text{les}} \cup \{l_t\} \). Before the agent executes the final action, the system first structures the action plan as a candidate path \( \hat{p}_{\text{final}} \), then queries the lesson memory for structurally similar entries \( L_{\text{rel}} \). If relevant lessons exist, a **deliberation loop** is triggered, forcing the agent to revise the plan:
\[
a_t \sim \pi'(\cdot|q_t, \mathcal{M}_{\text{val}}) =
\begin{cases}
\pi_{\theta}(\cdot|q_t, \mathcal{M}_{\text{val}}, L_{\text{rel}}) & \text{if } L_{\text{rel}} \neq \emptyset \\
\pi_{\theta}(\cdot|q_t, \mathcal{M}_{\text{val}}) & \text{otherwise}
\end{cases}
\]

This design transforms detected threats into adaptive defenses, enabling the agent not only to repel attacks but also to learn from them, progressively hardening its security posture.

## Experimental Setup

- **Tasks & Benchmarks** – Evaluated on three representative scenarios:  
  - Direct poisoning: knowledge‑intensive QA (ReAct‑StrategyQA) and healthcare record management (EHRAgent).  
  - Indirect injection: general QA based on MMLU.  
  - Multi‑agent systems: collaborative agents using the MISINFOTASK dataset.

- **Models & Baselines** – Tested on two backbone LLMs (GPT‑4o‑mini and LLaMA‑3.1‑8B) combined with two retrieval architectures (DPR and REALM). Baselines include: no defense, LLM auditor module, fine‑tuned Distil classifier, and perplexity (PPL) filter.

- **Metrics** – For direct poisoning, report attack success rates at retrieval (ASR‑r), agent thinking (ASR‑a), and end‑to‑end task performance (ASR‑t). For indirect injection, report final ASR. Utility is measured by accuracy (ACC) on benign tasks.

- **Key Hyperparameter** – Top‑k for both primary and lesson memories is set to 4.

## In‑Depth Analysis

A‑MemGuard not only provides an effective defense but also fundamentally redefines the paradigm of memory security for LLM agents – shifting from static filtering to a proactive, experience‑driven defense model.

**Methodologically**, the cleverness lies in not trying to “identify” malicious memories (which is nearly impossible in isolation), but instead indirectly detecting anomalies through the consensus of reasoning paths. This approach directly targets the Achilles’ heel of memory injection attacks – their effectiveness depends on specific contexts, and it is precisely in that context that anomalous reasoning reveals itself. In the authors’ words: “Although a single poisoned memory appears valid, the reasoning it induces will deviate from the consensus formed by benign experiences.”

**System‑wise**, the dual‑memory design embodies deep engineering wisdom. Traditional defenses are one‑shot – detect, block, and forget. A‑MemGuard, however, converts each defense experience into a reusable lesson, enabling continuous evolution of the defense system – particularly crucial against ever‑evolving attack vectors.

**Empirically**, the results are compelling: ASR‑r drops by over 97% in EHRAgent, and to 0.00% in ReAct‑StrategyQA for LLaMA‑3‑8B. Notably, these gains are consistent across multiple model backbones and retrieval architectures, affirming the method’s generalizability.

**One open question**: the effectiveness of consensus validation heavily relies on the majority of retrieved memories being benign. If an attacker can poison a sufficient number of entries to distort the consensus itself, how should we respond? This remains an intriguing direction for future research.

## Practical Applications

- **Enterprise LLM Agent Deployment** – A‑MemGuard can be integrated as a security middleware in customer service bots, internal knowledge assistants, etc., without retraining the model.

- **High‑Stakes Domains (Healthcare, Finance)** – In scenarios like medical record management (as tested with EHRAgent) or financial transaction analysis, the framework drastically reduces attack success rates while preserving high accuracy, offering direct business value.

- **Multi‑Agent Collaboration** – The paper validates scalability in multi‑agent systems, which is valuable for large‑scale automated workflows such as supply chain management or automated scientific research.

**Implementation Tips**:
1. Start with the open‑source code at [https://github.com/TangciuYueng/AMemGuard](https://github.com/TangciuYueng/AMemGuard) and test in a sandbox environment.
2. Tune the top‑k parameter (default 4) and the divergence threshold \( \tau \) based on your specific use case.
3. Monitor the growth of lesson memory; periodically review the quality of accumulated lessons to avoid over‑conservatism that could degrade utility.
4. For resource‑constrained environments, consider using a lighter model for reasoning path generation and comparison to reduce latency.

## References

- Original paper: Wei, Q., Yang, T., Wang, Y., Li, X., Li, L., Yin, Z., Zhan, Y., Holz, T., Lin, Z., & Wang, X. (2025). A-MemGuard: A Proactive Defense Framework for LLM-Based Agent Memory. *arXiv preprint arXiv:2510.02373*. [https://arxiv.org/abs/2510.02373](https://arxiv.org/abs/2510.02373)
- Open‑source code: [https://github.com/TangciuYueng/AMemGuard](https://github.com/TangciuYueng/AMemGuard)
