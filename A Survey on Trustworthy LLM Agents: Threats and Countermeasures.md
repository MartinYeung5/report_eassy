
# A Survey on Trustworthy LLM Agents: Threats and Countermeasures — In‑Depth Analysis

## Paper Highlights

This paper, published at KDD 2025, is a comprehensive survey on the trustworthiness of LLM‑based agents. It introduces the **TrustAgent** framework, which systematically divides the trustworthiness of LLM agents and multi‑agent systems (MAS) into **intrinsic** (brain, memory, tools) and **extrinsic** (users, agents, environment) dimensions. The survey thoroughly reviews attacks, defenses, and evaluation methods for each module, extending the traditional notion of “trustworthy LLM” to the new paradigm of “trustworthy agents”.

---

## Core Research Content

### Problem Definition

As LLMs evolve from standalone language models into **agents** equipped with memory, tool invocation, environmental interaction, and even multi‑agent collaboration, their attack surface expands significantly. Traditional trustworthiness research on single LLMs (e.g., adversarial examples, privacy leakage) can no longer cover the new threats arising from complex inter‑module interactions in agent systems. The core question is: **How can we systematically define, classify, and counter trustworthiness threats faced by LLM agents in real‑world deployment?**

### Innovative Approach

The **TrustAgent** framework innovates at three levels:

1. **Modular Taxonomy** – Deconstructs an agent system into three internal modules (Brain, Memory, Tools) and three external interaction dimensions (Users, other Agents, Environment). This modularisation provides clear focal points for threat analysis and defence design.

2. **Multi‑dimensional Connotations** – Refines trustworthiness into five dimensions: **Safety, Privacy, Truthfulness, Fairness, and Robustness**.

3. **Technical Implementation Orientation** – Systematically examines each module from three technical perspectives: **Attack, Defence, and Evaluation**.

### Research Outcomes

- **Attacks** – Systematically categorises attacks against the brain (prompt injection, backdoor attacks); against memory (poisoning, privacy leakage, misuse); and against tools (unauthorised invocation, adversarial example attacks).
- **Defences** – Summarises defence paradigms such as alignment, single‑model filtering, multi‑agent shields for the brain; retrieval filtering, prompt modification, and output intervention for memory.
- **Evaluation** – Points out that existing evaluation methods mainly rely on static datasets and cannot capture the complexity of dynamic agent–environment interactions.

### Potential for Practical Deployment

The framework offers significant engineering value. The authors have compiled all referenced studies according to their taxonomy and open‑sourced them in a GitHub repository (https://github.com/Ymm-cll/TrustAgent). This provides developers with a readily accessible threat checklist and a defence scheme reference table when designing and deploying LLM agent systems.

---

## Technical Details

### Overall Architecture of the TrustAgent Framework

```
┌─────────────────────────────────────────────────────────────┐
│                   TrustAgent Framework                      │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────┐    ┌─────────────────────────────┐ │
│  │   Intrinsic        │    │    Extrinsic                │ │
│  │   Trustworthiness  │    │   Trustworthiness           │ │
│  ├───────────────────┤    ├─────────────────────────────┤ │
│  │ • Brain           │    │ • User                     │ │
│  │ • Memory          │    │ • Other Agents             │ │
│  │ • Tools           │    │ • Environment              │ │
│  └───────────────────┘    └─────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  Trustworthiness Dimensions: Safety | Privacy | Truthfulness|
│                              Fairness | Robustness          │
├─────────────────────────────────────────────────────────────┤
│  Technical Perspectives: Attack | Defence | Evaluation     │
└─────────────────────────────────────────────────────────────┘
```

### Key Attack Techniques

**1. Attacks on the Brain Module**

The paper categorises brain attacks by manipulation mechanism:
- **Prompt Injection** – Attackers embed malicious instructions into the input to hijack the agent’s intended behaviour.
- **Backdoor Attacks** – For example, DemonAgent splits a backdoor into multiple sub‑backdoors with dynamic encryption; BLAST implements “infectious backdoors” that affect the reasoning of other agents after infecting a single one.

**2. Attacks on the Memory Module**

- **Memory Poisoning** – Injecting malicious data into the vector database. PoisonedRAG optimises malicious text through retrieval and generation conditions; AgentPoison attaches backdoor triggers to queries. Studies show that even a small amount of malicious information injected into the vector database can achieve a high attack success rate.
- **Privacy Leakage** – Attackers exploit the agent’s connection to long‑term memory to steal stored private data.

**3. Evaluation Metrics**

Key metrics mentioned include:
- **ASR (Attack Success Rate)** – measures attack effectiveness.
- **Precision, Recall, F1‑score** – for detecting malicious text retrieval.
- **CRR (Chunk Recovery Rate)** and **SS (Semantic Similarity)** – to measure the extent of data exfiltration from vector databases.

---

## Research Setup

### Scope and Methodology

This is a **survey paper**, not an experimental one. Its methodology includes:

1. **Systematic Literature Review** – Comprehensive collection of recent research on attacks, defences, and evaluations for LLM agent trustworthiness.
2. **Taxonomy Construction** – Proposes a new technical‑oriented taxonomy, updating old paradigms to the agent context.
3. **Cross‑module Analysis** – For each module (Brain, Memory, Tools), the paper follows a structured analysis: mechanism introduction → threat analysis → defence review → evaluation summary → future outlook.

### Supporting Resources

The paper provides an accompanying GitHub repository that organises all cited works according to the taxonomy, facilitating quick reference for researchers and practitioners.

---

## Comprehensive Analysis

### Core Insights

**1. “Agent ≠ LLM” – a different threat model**

The most important contribution is the clear statement that agent trustworthiness is far more complex than that of a standalone LLM. An LLM is a static model, whereas an agent is a dynamic system with memory, tools, and environmental interaction. This implies:
- The attack surface expands from “model input/output” to “memory retrieval chain, tool invocation chain, multi‑agent communication chain”.
- Defence shifts from “single‑point protection” to “system‑level protection”.

**2. Engineering value of modular thinking**

TrustAgent decomposes a complex agent system into three independent internal modules and three external dimensions. This modular thinking is particularly valuable for engineering practice – developers can inspect each module for security issues one by one, rather than facing a black box.

**3. Evaluation lags behind attacks**

The paper repeatedly notes the lack of systematic, reliable evaluation benchmarks. This reflects a common challenge: attack techniques advance quickly, but standardised evaluation systems are not yet established. The authors recommend building dedicated benchmarks for the attack and defence paradigms discussed.

**4. Dual nature of multi‑agent systems**

Multi‑agent systems (MAS) introduce both new threats (e.g., “infectious backdoors”) and new defence opportunities (e.g., multi‑agent collaborative shielding). This duality deserves further exploration.

### Limitations

As a survey, this paper focuses on “summarising and classifying” rather than proposing new solutions. Readers seeking direct “how‑to” guidelines for hardening their agent systems may need to consult the primary sources cited in the paper.

---

## Practical Applications

### Recommendations for Agent Developers

**1. Design Phase: Threat Modelling**

When architecting an agent system, perform systematic threat modelling using the TrustAgent taxonomy:
- Brain: consider prompt injection and backdoor risks.
- Memory: assess vector database poisoning and privacy leakage.
- Tools: review permission controls on tool invocation interfaces.
- Interactions: analyse trust propagation in multi‑agent communication.

**2. Development Phase: Multi‑layer Defence**

- **Input side** – deploy prompt filters (e.g., StruQ, ShieldLM).
- **Memory side** – implement retrieval isolation and aggregation strategies (e.g., RobustRAG’s Isolate‑then‑Aggregate).
- **Output side** – add safety filters to block unsafe token generation.
- **System side** – consider deploying a guard agent as an independent security layer.

**3. Evaluation Phase: Dynamic Testing**

Since static datasets cannot cover dynamic agent–environment interactions, it is advisable to:
- Design dynamic evaluation mechanisms that test agents against real‑time attacks in simulated environments.
- Build continuous learning‑style evaluation frameworks that test agents on evolving data streams.

**4. Operations Phase: Continuous Monitoring**

- Periodically audit the memory system for malicious injections.
- Monitor abnormal communication patterns in multi‑agent systems.
- Establish privacy protection mechanisms (e.g., query rewriting, LLM fine‑tuning) to prevent privacy leakage.

---

## References

- Original paper: Yu, M., Meng, F., Zhou, X., Wang, S., Mao, J., Pang, L., Chen, T., Wang, K., Li, X., Zhang, Y., An, B., & Wen, Q. (2025). A Survey on Trustworthy LLM Agents: Threats and Countermeasures. In *Proceedings of the 31st ACM SIGKDD Conference on Knowledge Discovery and Data Mining V.2* (pp. 6216–6226). ACM. https://doi.org/10.1145/3711896.3736561

- arXiv version: https://arxiv.org/abs/2503.09648

- Companion GitHub repository: https://github.com/Ymm-cll/TrustAgent
