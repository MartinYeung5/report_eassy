# MINJA: A Practical Memory Injection Attack against LLM Agents (2025)

## Key Insights

MINJA (Memory INjection Attack) enables memory poisoning without directly tampering with an agent’s memory bank. By interacting only through normal user queries, an attacker can induce the LLM agent to autonomously generate and store malicious records, which later mislead the agent’s reasoning when a victim query arrives. The paper validates this attack across healthcare, e-commerce, and general Q&A scenarios, revealing a critical security flaw in the memory management mechanisms of current LLM agents.

---

## Core Research

### Problem Definition

LLM agents differ from vanilla LLMs in that they are equipped with **long-term memory**: the system stores past queries and reasoning traces, and when a new query comes in, it retrieves the most relevant historical records as in‑context demonstrations to assist task completion.

However, this design introduces a severe security risk: **if the memory is poisoned, the retrieved malicious examples can mislead the agent’s reasoning**. The paper gives a disturbing example – if an autonomous driving agent’s memory is injected with a record like “perform emergency braking at extremely high speed”, a user might encounter sudden stops on the highway, causing fatal accidents.

Previous work (e.g., AgentPoison) explored similar threats, but assumed the attacker has **direct write access** to the memory bank – an unrealistic privilege in most real‑world systems. MINJA addresses the core question: **Can an attacker, with only ordinary query‑level interaction, poison the memory of an LLM agent?**

### Innovative Method

MINJA’s design tackles two fundamental challenges:

**Challenge 1: How to craft a malicious record that effectively misleads the agent?**

The attacker aims that for a victim query \( q_v \) (containing victim entity \( v \)), the agent produces the reasoning steps \( R_{q_t} \) corresponding to a target query \( q_t \) (where \( v \) is replaced with target entity \( t \)).

The problem is that there is a **logical gap** between \( q_v \) and \( R_{q_t} \) – e.g., why should a query about “patient A” lead to reasoning about “patient B”?

MINJA introduces **Bridging Steps** – e.g., in the medical scenario, a bridging step could be “patient A’s data is now stored under patient B’s name”. This plausible intermediate reasoning connects the victim query to the target reasoning. The bridging steps must be general enough, since victims may ask arbitrary queries containing \( v \). Also, the malicious record’s query must appear benign to ensure it is retrieved later.

**Challenge 2: How to induce the agent to generate and store these malicious records itself?**

Since the attacker cannot write directly to memory, the agent must be tricked into doing it. MINJA designs an **Indication Prompt** – a sequence of reasoning steps appended to the attack query, which guides the agent to generate the bridging steps.

But another issue arises: if the query always contains this indication prompt, the stored record may have low similarity to normal victim queries, reducing retrieval probability. To solve this, MINJA proposes a **Progressive Shortening Strategy (PSS)** – gradually removing parts of the indication prompt from the query, while ensuring the agent’s response still contains the bridging steps and target reasoning. After multiple iterations, the final stored record \((a_v, [b_{v,t}, R_{a_t}])\) appears indistinguishable from normal records.

In summary, the attack flow: attacker submits query with indication prompt → agent responds with bridging steps and stores them → attacker shortens the prompt progressively → eventually a “clean” malicious record is stored → when a victim query arrives, that record is retrieved as an example → the agent is misled to generate the target reasoning.

### Research Results

The paper evaluates on three different agent types:

| Agent | Scenario | Dataset | Base Model |
|-------|----------|---------|------------|
| EHRAgent | Healthcare (EHR) | MIMIC‑III, eICU | GPT‑4 |
| RAP | E‑commerce (Webshop) | Webshop | GPT‑4, GPT‑4o |
| QA Agent | General QA | MMLU | GPT‑4, GPT‑4o |

Three metrics are used:
- **Injection Success Rate (ISR)**: proportion of malicious records successfully stored
- **Attack Success Rate (ASR)**: proportion of stored malicious records that successfully mislead the agent
- **Utility Drop (UD)**: impact on the agent’s normal task performance

Key results:
- **ISR generally > 90%**: EHRAgent reached 95.6% on MIMIC‑III, 98.5% on eICU; RAP (GPT‑4o) 99.3%; QA Agent 100%.
- **ASR is prominent**: EHRAgent 90.0% on eICU; RAP (GPT‑4o) 98.5%; ASR varied between 57% and 98.5% across different scenarios.
- **UD is minimal**: in almost all cases UD is within ±10%, indicating negligible side effects on normal functionality.

Notably, the paper was accepted as a NeurIPS 2025 Poster, attesting to its academic significance.

### Real‑world Applicability

MINJA’s attack conditions are extremely relaxed – the attacker only needs to interact as a normal user, without any system privileges. This means:

- **Any user with access to a public agent service could become an attacker.**
- Shared memory banks are common in existing agent frameworks (e.g., ChatGPT’s “improve the model for everyone” feature).
- Even with isolated memory, account impersonation is far more practical than obtaining system‑level write access.

From a defensive perspective, the vulnerabilities exposed by MINJA are urgent. As LLM agents are increasingly deployed in high‑stakes domains like healthcare, finance, and autonomous driving, the potential harm of such attacks cannot be ignored.

---

## Technical Details

### Formal Definition of the Attack

Let the victim query be \( q_v \), containing a victim entity \( v \). The attacker’s goal is that for \( q_v \), the agent produces the target reasoning \( R_{q_t} \), where \( q_t \) is the query with \( v \) replaced by target entity \( t \).

The ideal form of the malicious record is:

\[
(a_v, [\mathbf{b}_{v,t}, \mathbf{R}_{a_t}])
\]

where:
- \( a_v \): attack query containing victim entity \( v \) (appears benign)
- \( \mathbf{b}_{v,t} \): bridging steps that logically connect \( v \) and \( t \)
- \( \mathbf{R}_{a_t} \): reasoning for the target query \( a_t \)

### Progressive Shortening Strategy (PSS)

Let the indication prompt be \( [r_1, r_2, \ldots, r_n] \). Each shortening step is:

\[
[a_v, r_1, r_2, \ldots, r_{n-i}] \rightarrow [a_v, r_1, r_2, \ldots, r_{n-i-1}]
\]

i.e., removing one step at a time from the end. After \( n \) iterations, the stored record becomes \( (a_v, [b_{v,t}, R_{a_t}]) \).

### Retrieval Mechanism

The agent retrieves the \( k \) most similar records from memory based on **query similarity** – typically using cosine similarity on text embeddings. MINJA exploits this by making the attack query \( a_v \) highly similar to the victim query \( q_v \), ensuring the malicious record is retrieved.

---

## Experimental Setup

### Hardware and Software

The paper does not specify exact hardware, but from the description we infer:

- **Models**: GPT‑4 and GPT‑4o (via API)
- **Embedding models**: all‑MiniLM‑L6‑v2 (for EHRAgent and RAP), text‑embedding‑ada‑002 (for QA Agent)
- **Retrieval number**: RAP retrieves 3, EHRAgent 4, QA Agent 5

### Design Details

- **Victim‑Target pairs**: 9 independent experiments per configuration
  - Healthcare: Patient ID pairs, Medication pairs
  - E‑commerce: Item pairs
  - QA: Term pairs (answers shifted by 4 letters alphabetically)
- **Number of attack queries**: 10 for MMLU, 15 for others
- **Shortening iterations**: 4 for Patient ID, 5 for Medication/Items/Terms
- **Initial memory**: EHRAgent pre‑filled with 4 benign records; RAP and MMLU start empty

---

## Comprehensive Analysis

MINJA does more than just “find a vulnerability” – it pushes the frontier of LLM agent security research on several levels.

**First, a breakthrough in threat modelling.** Previous memory attacks assumed direct write access – an unrealistic premise. MINJA reduces the attacker to an ordinary user, revealing that **even without any privileges, memory poisoning is still feasible**. That is the truly unsettling takeaway.

**Second, the technical ingenuity.** Bridging steps + Indication Prompt + Progressive Shortening form a cohesive chain. Bridging steps resolve the logical gap; Indication Prompt induces the agent to generate; Progressive Shortening removes traces. The combination ensures both effectiveness and stealth.

**Third, the breadth of experimentation.** The paper does not stop at a single domain but covers healthcare, e‑commerce, and general Q&A – showing the **generality** of the flaw. It is not a specific implementation bug but a systemic risk inherent in the “shared memory + similarity retrieval + in‑context learning” architecture.

**One notable detail is the variance in ASR.** On MMLU, ASR ranges from 40% to 100%, with a standard deviation of 19.1%. This suggests that some victim‑target pairs are easier to mislead than others. This instability poses challenges for attackers, but also hints at potential defences – perhaps by analysing which entity substitutions are more “bridgeable”.

**From a broader perspective,** MINJA exposes a **breakdown of trust**. The agent trusts that historical records in memory are reliable, but MINJA proves this trust can be easily exploited. This is reminiscent of the classic “trusting input” problem in software security – and for LLM agents, we might add a new principle: **never unconditionally trust historical records in memory**.

---

## Practical Implications

### For Agent Developers

1. **Memory isolation**: The most straightforward defence is to avoid sharing memory across users. If sharing is necessary, at least enforce strict auditing of write operations.
2. **Retrieval filtering**: Before using a record as an in‑context example, add a safety check – detect anomalous “bridging” logic (e.g., mapping entity A to entity B).
3. **Be cautious with storing chain‑of‑thought**: The paper notes that storing full CoT traces increases manipulation surface – consider storing only final outcomes.
4. **Write‑time policy**: Perform anomaly detection on records about to be written, flagging potential bridging steps or unusual entity mappings.

### For End Users

- When using public agent services (e.g., ChatGPT, medical chatbots), be aware that the output you receive **may have been influenced by interactions from other users**.
- In high‑stakes decisions (medical, financial, safety), treat agent outputs with caution and seek human verification when possible.

### For Researchers

- **Defence mechanisms**: How to effectively detect and mitigate MINJA‑style attacks? This remains largely open.
- **Further attack evolution**: MINJA targets entity substitution; can more complex manipulations (e.g., sentiment shifts, stance changes) be similarly achieved?
- **Formal security models**: Can we develop a rigorous framework for memory security in agent systems to guide safe design?

---

## References

- Original paper: Dong, S., Xu, S., He, P., Li, Y., Tang, J., Liu, T., Liu, H., & Xiang, Z. (2025). *Memory Injection Attacks on LLM Agents via Query‑Only Interaction*. arXiv:2503.03704. [https://arxiv.org/abs/2503.03704](https://arxiv.org/abs/2503.03704)
- NeurIPS 2025 Poster: [https://neurips.cc/virtual/2025/loc/san-diego/poster/118152](https://neurips.cc/virtual/2025/loc/san-diego/poster/118152)
- Code repository: The paper notes that code is open‑sourced (see footnote on the first page).
