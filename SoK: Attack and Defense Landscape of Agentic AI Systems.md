
# SoK: Attack and Defense Landscape of Agentic AI Systems —— Deep Academic Analysis

## Key Points

This paper, co-authored by Juhee Kim (UC Berkeley & Seoul National University), Wenbo Guo (UC Santa Barbara), and Dawn Song (UC Berkeley), is a Systematization of Knowledge (SoK) published at USENIX Security 2026. It is **the first comprehensive, systematic survey dedicated to the security of Agentic AI systems**. The work systematically analyzes the design space, attack surface, and defense mechanisms for these systems, establishing the first structured security research framework in this emerging field.

---

## Core Research Content

### Problem Definition

Agentic AI systems – **hybrid architectures that deeply integrate Large Language Models (LLMs) with non‑AI software components (tools, APIs, file systems, network interfaces, etc.)** – are rapidly entering real‑world applications, from intelligent chatbots to software development automation and web browsing agents. However, this unprecedented flexibility introduces **fundamentally different and complex security challenges** compared to traditional software systems.

The core issue is that traditional security models assume deterministic and predictable system behaviour, whereas Agentic AI systems possess **autonomy, goal‑driven reasoning, planning capabilities, and the ability to take actions in digital or physical environments**. The gap between the probabilistic nature of LLMs and the deterministic execution of tools dramatically expands the attack surface, requiring a fundamental rethinking of threat models. Specifically:

1. **Blurred trust boundaries**: An agent acts simultaneously as a “process”, a “data store”, and an “actor”, making traditional STRIDE threat modelling difficult to apply.
2. **Diverse attack vectors**: From prompt injection to tool poisoning, knowledge‑base poisoning, and emergent multi‑agent threats, the attack surface goes far beyond that of conventional LLM applications.
3. **Lack of defensive evaluation frameworks**: Existing defences perform poorly against adaptive attacks, and there is no unified evaluation standard or benchmark.

### Innovative Methodology

The innovation of this paper lies in its systematic methodological breakthroughs:

- **(1) First systematic framework**: This is the **first SoK in the academic community to systematically address Agentic AI security**. It builds a complete research framework from three dimensions – design space, attack surface, and defence mechanisms. The paper compiles **128 representative papers, 51 attack types, and 60 defence mechanisms**, covering work from top security conferences (USENIX Security, NDSS, etc.) and top ML conferences (NeurIPS, ICLR, ICML, etc.), as well as high‑impact preprints, industry white papers, and CVE reports.

- **(2) Design‑space analysis**: The paper proposes **seven core design dimensions** that determine the attack surface, systematically analysing how different design choices affect the security posture of the system.

- **(3) Comprehensive attack‑surface mapping**: The paper constructs a complete attack taxonomy covering **prompt‑layer injection, knowledge‑base poisoning, tool/plugin exploitation, and emergent multi‑agent threats**.

- **(4) Case‑study approach**: Through concrete case studies, the paper reveals **systematic gaps** in current security practices, rather than merely enumerating vulnerabilities.

### Research Results

The core findings include:

1. **First systematic security framework**: Provides a theoretical foundation for understanding the security risks and defence strategies of AI agents.

2. **Comprehensive attack taxonomy**: Systematically categorises **51 attack types**, covering everything from prompt injection to tool poisoning, and from single‑agent to multi‑agent collaborative scenarios.

3. **Systematised defence mechanisms**: Organises **60 defence mechanisms** and critically evaluates their effectiveness.

4. **Identification of key gaps**: Highlights the grim reality that **adaptive attacks consistently achieve bypass rates exceeding 50%** against current defences, exposing fundamental deficiencies in existing defensive research.

5. **Open challenges**: Identifies open research questions for this emerging field.

### Practical Deployment Potential

**High feasibility**:

- **Security auditing and risk assessment**: Organisations can use the framework to conduct comprehensive security audits of their Agentic AI systems, identifying design‑level security flaws.
- **Integration into secure development lifecycles**: Incorporate the seven design dimensions into the early design phase of Agentic AI systems to achieve “shift‑left” security.
- **Red‑teaming exercises**: Build systematic red‑team test plans based on the 51‑attack‑type taxonomy.

**Medium feasibility**:

- **Standardisation**: Provide a theoretical basis for industry standards (e.g., ISO, NIST) regarding Agentic AI security.
- **Security product development**: Develop targeted security tools based on the analysis of 60 defence mechanisms.

**Challenges and limitations**:

- The paper is a **survey study** rather than an empirical one; specific defence solutions require further technical development.
- Agentic AI technology itself is evolving rapidly, so the security framework needs continuous updating.

---

## Technical Details

### Core Architecture Analysis

A typical Agentic AI system architecture can be abstracted into the following layers:

```
┌─────────────────────────────────────────────┐
│            Governance Layer                  │
├─────────────────────────────────────────────┤
│         Multi‑Agent Coordination Layer       │
├─────────────────────────────────────────────┤
│           Tool Execution Layer               │
├─────────────────────────────────────────────┤
│              Memory Layer                    │
├─────────────────────────────────────────────┤
│            Cognitive Layer                   │
├─────────────────────────────────────────────┤
│          Foundation Model Layer              │
└─────────────────────────────────────────────┘
```

Each layer has its own unique security threats, and threats can propagate across layers.

### Key Attack Vectors

The systematised attack types include, among others:

1. **Prompt Injection**
   - Direct injection: malicious instructions inserted directly into user input.
   - Indirect injection: injected through external data sources (web pages, documents, emails).

2. **Knowledge‑base Poisoning**
   - Contaminating the RAG (Retrieval‑Augmented Generation) knowledge base of the agent.
   - Memory‑layer attacks can produce effects indistinguishable from model failures.

3. **Tool/Plugin Exploits**
   - Tool poisoning: hijacking agent behaviour through malicious tool outputs.
   - Privilege escalation: exploiting excessive permissions to perform unauthorised actions.

4. **Emergent Multi‑Agent Threats**
   - Malicious collaboration between agents.
   - Propagation of attack payloads through inter‑agent communication.

### Trust Boundary Model

The paper argues that the root cause of security challenges in Agentic AI systems is the **emergence of probabilistic trust boundaries**. In traditional systems, trust boundaries are deterministic (e.g., process boundaries, network boundaries). However, an agent’s decisions are based on probabilistic outputs from an LLM, making the trust boundary fuzzy and dynamically changing. This **trust‑authorisation mismatch** is the fundamental cause of most security threats.

---

## Research Setup

### Methodological Design

As an SoK paper, the research design primarily consists of:

- **Literature scope**: Covers top security conferences (USENIX Security, NDSS, IEEE S&P, etc.) and top ML conferences (NeurIPS, ICLR, ICML, etc.), as well as high‑impact preprints, industry white papers, and CVE reports.
- **Analysis dimensions**: Design space → Attack surface → Defence mechanisms → Gap identification → Open challenges.
- **Taxonomy approach**: Threat classification based on system architecture layers, rather than merely enumerating attack types.

### Hardware/Software Configuration

As a survey, the paper does not require specific hardware or software. However, for researchers and practitioners who wish to **reproduce or apply** its framework, the following are recommended:

- **LLM deployment environment**: Inference environment supporting mainstream LLMs (GPT series, Claude, Llama, etc.).
- **Agent frameworks**: LangChain, AutoGPT, BabyAGI, and other mainstream agent development frameworks.
- **Security testing tools**: Prompt injection testing tools, permission auditing tools, etc.
- **Computing resources**: Depends on the scale and complexity of the specific agent system.

---

## Comprehensive Analysis

### Academic Contributions

The academic value of this paper is reflected in several aspects:

**Filling a gap**: Agentic AI was one of the most active research areas between 2024 and 2026, but prior to this work, there was no systematic security survey. This paper is the first to systematically integrate fragmented knowledge scattered across security conferences, ML conferences, and industry practice.

**Methodological innovation**: The paper does not stop at a simple literature catalogue; it builds a three‑dimensional analytical framework of **design space → attack surface → defence mechanisms**. This structured approach enables researchers to:
- Understand the security implications of different design choices.
- Anticipate possible new forms of attack.
- Evaluate the applicability boundaries of defence mechanisms.

**Interdisciplinary bridge**: The paper connects the security community and the ML community – two groups that often have significantly different perspectives on Agentic AI security issues.

### Practical Insights

The finding that **“adaptive attacks achieve bypass rates >50%”** carries profound implications: most current defences are trained on known attack patterns and degrade sharply against attackers who can adapt their strategies in real time. This suggests that:

1. Current defence research may **overfit to specific attack patterns**.
2. A paradigm shift from “signature matching” to “behavioural anomaly detection” is needed.
3. Red‑teaming **must employ adaptive attack strategies** to reflect real‑world threats.

### Limitations and Critical Reflections

1. **Timeliness challenge**: The Agentic AI field evolves extremely rapidly; new attack surfaces and defence techniques may emerge after the paper’s publication.
2. **Lack of empirical validation**: As an SoK, the paper does not provide large‑scale empirical comparisons of different defence mechanisms in real‑world scenarios.
3. **Coverage of industry practice**: Although industry white papers are included, the security practices of closed‑source commercial systems may not be fully reflected.
4. **Concretisation of “design dimensions”**: The detailed contents and interrelationships of the seven design dimensions are not fully disclosed in the public abstract; access to the full paper is needed.

---

## Practical Applications

### Recommendations for Researchers

1. **Identify research gaps**: Use the paper’s attack taxonomy and defence analysis to pinpoint under‑researched attack vectors or defence strategies.
2. **Build evaluation benchmarks**: Develop standardised security evaluation benchmarks covering the 51 attack types based on the framework.
3. **Focus on root causes**: The “trust‑authorisation mismatch” identified by the paper is a fundamental issue worth deeper investigation.

### Recommendations for Engineers/Architects

1. **Security considerations during design**: Incorporate the seven design dimensions into threat modelling at the early design stage of Agentic AI systems.
2. **Layered defence**: Deploy targeted defences at the foundation model, cognitive, memory, tool execution, and multi‑agent coordination layers.
3. **Assume breach**: Given the high bypass rate of adaptive attacks, assume that defences may be breached, and implement defence‑in‑depth and rapid response mechanisms.
4. **Least privilege**: Strictly limit the tool access permissions of agents to avoid over‑authorisation.

### Recommendations for Security Teams

1. **Upgrade red‑teaming**: Use adaptive attack strategies rather than only testing known attack patterns.
2. **Continuous monitoring**: Build anomaly detection capabilities focusing on agent behaviour, rather than relying solely on static defences.
3. **Supply‑chain security**: Pay attention to supply‑chain risks arising from third‑party tools, models, and knowledge bases that agents depend on.

---

## References

- Original paper page (USENIX Security 2026): https://www.usenix.org/conference/usenixsecurity26/presentation/kim-juhee-agentic
- arXiv preprint: https://arxiv.org/abs/2603.11088
- SoK paper list (Oakland SoK): https://oaklandsok.github.io/usenix/
- Authors: Juhee Kim (UC Berkeley/Seoul National University), Wenbo Guo (UC Santa Barbara), Dawn Song (UC Berkeley)

---
