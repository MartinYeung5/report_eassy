# The Emerged Security and Privacy of LLM Agents: A Survey with Case Studies

## Paper Highlights

This paper is a systematic survey on the security and privacy threats facing large language model (LLM) agents. It systematically categorizes the sources of threats—both vulnerabilities inherited from underlying LLMs and new threats unique to agents—and analyzes their real-world impact on users, environments, and other agents. The survey also summarizes existing defense strategies and outlines future research directions.

---

## Core Research Content

### Problem Definition

LLM agents are now widely deployed in virtual assistants, customer service bots, educational tools, and many other scenarios, handling massive data and interacting with humans. However, as their commercial value and application scope expand, they have also revealed severe security and privacy vulnerabilities. Unlike static LLMs, LLM agents possess dynamic capabilities—their immediate responses influence future decisions and actions, leading to broader risks. Current research on LLM agents is still in its early stages, with most studies focusing on attacks against LLMs themselves, while a comprehensive overview of the more complex security and privacy issues at the agent level remains lacking. This survey aims to fill that gap.

### Innovative Approach

The paper's innovation lies in constructing a systematic threat analysis framework that divides the security threats facing LLM agents into two major categories:

**1. Threats inherited from LLMs**, further subdivided into:
- **Technical vulnerabilities**, including hallucination, catastrophic forgetting, and misunderstanding, which originate from data and algorithm limitations during model training.
- **Malicious attacks**, such as data theft and response tampering (e.g., data extraction attacks and a series of instruction-tuning attacks).

**2. Threats unique to agents**, organized based on the LLM agent workflow (Thought, Action, Perception):
- **Knowledge poisoning**: contaminating training data and knowledge bases.
- **Output manipulation**: interfering with the agent's thought and perception phases.
- **Functional manipulation**: exploiting agent interfaces and tools to perform unauthorized operations.

Additionally, the paper uses a concrete case study—an agent named "Eva" in a virtual town—to vividly illustrate the practical manifestations of these threats.

### Research Outcomes

As a survey, the main contributions are:

- **A threat taxonomy**: systematically identifying and categorizing emerging threats to LLM agents, distinguishing inherited and agent-specific threats.
- **An impact analysis framework**: elaborating on the real-world impact of various threats on humans, the environment, and other agents.
- **A review of defense strategies**: summarizing existing mitigation measures and solutions.
- **Future research directions**: identifying current gaps and proposing promising avenues for further investigation.

### Practical Deployment Potential

The paper's value extends beyond academia, offering significant guidance for practical applications:

- **Security by design during development**: developers can use the threat taxonomy for risk assessment and preemptive architectural safeguards.
- **Pre‑deployment security testing**: enterprises can conduct specialized tests for inherited threats (e.g., hallucination detection) and agent‑specific threats (e.g., functional manipulation) before deployment.
- **Runtime monitoring and defense**: the reviewed defense strategies provide a theoretical basis for runtime security monitoring.
- **Industry compliance and standard setting**: as LLM agents permeate various sectors, the findings can inform the development of relevant security standards and regulatory frameworks.

---

## Technical Details

### Core Architecture of LLM Agents

LLM agents are complex AI systems built upon foundation models such as GPT‑4, Claude 3, and Llama 3. Their core capabilities include natural language understanding and generation, decision‑making, problem‑solving, and human‑like interaction. The paper summarises the agent workflow into three key phases:

1. **Thought**: the agent reasons and plans based on the input.
2. **Action**: the agent executes specific operations or invokes external tools.
3. **Perception**: the agent receives and interprets environmental feedback.

### Key Technical Vulnerabilities

**Hallucination**: the phenomenon where model outputs are inconsistent or unreliable with respect to the input or source content. Causes include:
- Nature of training data (containing misinformation and biases)
- Model architecture (e.g., limited representational capacity and attention mechanisms)
- Decoding strategies (e.g., increased randomness can exacerbate hallucination)

**Catastrophic Forgetting**: when LLMs are fine‑tuned on small, specific datasets, they overfit to the new data and lose performance on other tasks. Studies show that larger models tend to suffer more severe forgetting, and introducing more tasks during continual instruction tuning usually leads to more pronounced forgetting.

**Misunderstanding**: the agent fails to fully comprehend or accurately respond to human instructions or intents during interaction. Contributing factors include pre‑training data characteristics, task‑specific settings, and interaction contexts.

### Malicious Attack Types

The paper highlights several malicious attacks against LLMs:
- **Data extraction attacks**: stealing training data from the model.
- **Instruction‑tuning attacks**: crafting malicious instructions to induce harmful outputs.
- **Jailbreaking**: bypassing safety and censorship mechanisms to generate restricted content.

---

## Research Setup

### Case Study Design

The paper employs a virtual town scenario for its case study:

- **Environment**: the town includes various real‑life locations (shops, offices, restaurants, museums, parks, etc.).
- **Agent roles**: each LLM agent acts as an independent resident, playing distinct roles and performing different functions, mimicking human behaviour in a community.
- **Operation modes**: agents can be manually controlled to complete tasks with specific characters, or run autonomously—following their own plans and acquiring new knowledge through community interactions.
- **Example agent**: a store employee named "Eva" with capabilities including:
  - Real‑time parsing of customer queries and responding accordingly
  - Integrating with an inventory management system via APIs to automatically track stock levels
  - Employing advanced reasoning to assist customers with purchasing decisions
  - Generating personalised promotional emails based on promotions and customer history
  - Independently managing shelf stock and price tag updates
  - Handling returns or complaints and escalating to human management when needed

### Hardware and Software Requirements (Implicit)

Although not explicitly listed, the following can be inferred:

- **Base model**: a GPT‑4‑level or equivalent large language model as the core controller.
- **API integration**: interfaces for interacting with external systems and tools.
- **Data storage and knowledge base**: infrastructure for large‑scale data storage and retrieval.
- **Security monitoring**: runtime mechanisms for detecting and defending against attacks.

---

## Comprehensive Analysis

### Dual Sources of Threats and Their Interaction Amplification

A key insight of this paper is the "dual‑source" nature of LLM agent security threats and their mutual reinforcement. Technical vulnerabilities provide opportunities for malicious attackers, while malicious attacks further amplify the risks caused by vulnerabilities. This interaction makes the security challenges of LLM agents far more complex than those of traditional software systems.

### From Static to Dynamic: A Paradigm Shift in Security

Traditional LLM security research focuses on the input‑output security of static models. LLM agents, however, introduce a new dimension of dynamics—an agent's immediate response affects its future decisions and actions. A single successful attack can produce cascading effects, altering the agent's long‑term behaviour patterns. This temporal dimension of risk propagation demands proactive and continuous defense strategies.

### Extension of Risk from Cyberspace to Physical Space

The paper emphasises that when LLM agents are integrated into physical entities like robots, attacks can pose serious threats to physical safety, financial security, or overall system integrity. This marks a tangible extension of AI security risks from the digital realm to the physical world, with potentially severe consequences.

### Gaps in Current Research

The paper candidly acknowledges that research on LLM agents is still nascent. Most existing studies concentrate on attacks against LLMs themselves, while comprehensive research on agent‑specific security and privacy issues remains scarce. This gap underscores the academic and practical significance of this survey.

---

## Practical Applications

### Recommendations for Agent Developers

1. **Threat modelling upfront**: incorporate the proposed threat taxonomy early in the design phase for systematic risk assessment.
2. **Data security hardening**: implement strict data cleansing and validation processes to guard against knowledge poisoning.
3. **Tool‑call security**: for functional manipulation threats, enforce the principle of least privilege on external tools and APIs, and validate inputs/outputs.
4. **Output auditing**: establish real‑time review and filtering mechanisms to counter output manipulation.

### Recommendations for Enterprises Deploying Agents

1. **Phased deployment**: pilot in low‑risk scenarios first, then gradually expand to higher‑risk applications.
2. **Continuous monitoring and incident response**: set up real‑time monitoring for technical vulnerabilities like hallucination, and prepare incident response plans.
3. **User education and transparency**: clearly communicate the agent's capabilities and limitations to users to prevent over‑reliance.
4. **Periodic security assessments**: regularly evaluate deployed agents against the threat taxonomy and conduct penetration testing.

### Recommendations for Researchers

1. **Innovative defenses**: develop novel defense mechanisms targeting agent‑specific threats.
2. **Multi‑agent security**: investigate how threats propagate and how to defend in multi‑agent systems.
3. **Human‑agent interaction security**: explore the broader socio‑technical issues in human‑agent interactions, requiring interdisciplinary research.

---

## References

- Original paper: He, F., Zhu, T., Ye, D., Liu, B., Zhou, W., & Yu, P. S. (2024). The Emerged Security and Privacy of LLM Agent: A Survey with Case Studies. *arXiv preprint arXiv:2407.19354v1*. [https://arxiv.org/html/2407.19354v1](https://arxiv.org/html/2407.19354v1)
