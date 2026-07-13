# Context Rules! Privacy and Security for Future Trustworthy AI Agents

## Paper Highlights

This paper argues that judging the safety of an AI agent’s actions **must depend on context** – the same operation can be perfectly legitimate in one situation and completely违规 in another. To address this, the authors propose **Conseca (Contextual Agent Security)**, a framework that leverages large language models to dynamically generate “instant, context‑aware, and human‑verifiable” security policies, fundamentally solving the core contradiction that traditional static rules cannot adapt to the vast number of real‑world scenarios faced by general‑purpose AI agents.

---

## Core Research Content

### Problem Definition

Traditional computer security relies on **human‑predefined static policies** – e.g., mobile app permissions, network access control lists. These approaches work when the application scope is limited and agent autonomy is constrained. However, when general‑purpose AI agents (e.g., automated personal assistants) are deployed in real environments supporting **a massive diversity of tasks**, the limitations of static policies become glaring:

- Policy writers **cannot exhaustively enumerate all possible contexts**, leading to rules that are either overly restrictive in some scenarios or overly permissive in others.
- Relying on manual user confirmation leads to **user fatigue and overpermissioning**.
- The same action (e.g., “delete an email”) has completely different security implications depending on context – the email content, the user’s goal (clearing sensitive emails vs. cleaning spam), the mailbox type (work vs. personal), etc.

### Innovative Approach

The core innovation is the **Conseca (Contextual Agent Security) framework**, whose key ideas are:

1. **Introduce contextual security to the AI agent domain**: Borrowing from the well‑established privacy theory of **Contextual Integrity**, the framework systematically applies contextualisation to agent security.
2. **Leverage LLMs for dynamic policy generation**: Instead of relying on hand‑written static rules, Conseca uses an LLM at runtime to **instantaneously generate** security policies that match the current specific context.
3. **Human‑verifiable policies**: The generated policies are not black‑box outputs; they are designed to be **human‑understandable, auditable, and intervenable**, ensuring controlled security decisions.

### Research Outcomes

Conseca was presented at the ACM HotOS 2025 workshop. Its main outcomes include:

- **Problem formalisation**: Systematic demonstration of why traditional security paradigms fail in the era of general‑purpose AI agents, and why context must become a core dimension of security decisions.
- **Framework design**: Conceptual architecture and operational mechanisms of Conseca, showing how LLMs can enable fine‑grained “one policy per scenario” security control.
- **Foundation for follow‑up work**: This work lays the theoretical groundwork for subsequent studies – for instance, later research on prompt injection attacks shows that contextual approaches like Conseca can dynamically protect agents, though they are still early in fully determining contextual appropriateness.

### Real‑World Application Potential

Conseca has broad and practical deployment prospects:

- **Automated personal assistants** – such as Google Assistant or Siri, when handling cross‑app operations, can dynamically determine permissions based on the current dialogue context.
- **Enterprise AI Copilots** – in code generation, document processing, etc., security boundaries can be adjusted dynamically according to task sensitivity and user roles.
- **Multi‑agent systems** – combined with the team’s concurrently released **Terrarium framework**, Conseca enables fine‑grained security and privacy research in multi‑agent interactions.
- **Privacy protection** – the team’s **AirGapAgent** (published at CCS’24) has already demonstrated the effectiveness of similar ideas: on the Gemini Ultra agent, a single‑query context‑hijacking attack dropped its privacy protection rate from 94% to 45%, while AirGapAgent achieved a 97% protection rate.

---

## Technical Details

### Core Mechanism of Conseca

The workflow of the Conseca framework can be summarised in the following key steps:

**1. Context awareness**: The system captures the complete context of the current interaction in real time, including but not limited to:
- User identity and role
- Task goal and intent
- Attributes of the operation target (e.g., file type, sensitivity level)
- Environmental state and historical interaction records

**2. Just‑in‑time policy generation**: An LLM dynamically generates security policies based on the captured context. This process differs from traditional rule‑matching – the model “understands” the current situation and reasons about appropriate security boundaries.

**3. Policy verification and enforcement**: The generated policies must satisfy two critical conditions:
- **Context matching**: The policy must precisely correspond to the specific current context, not a generic rule.
- **Human‑verifiability**: Policies are presented in a human‑understandable form, allowing manual review and intervention.

### Theoretical Foundation – Contextual Integrity

Conseca is rooted in Nissenbaum’s **Contextual Integrity** privacy theory. That theory holds that the appropriateness of information flow depends on three core dimensions:
- **Actors**: who sends and who receives the information
- **Information types**: what kind of information is being transmitted
- **Transmission principles**: what norms govern the flow

Conseca extends this framework from privacy to **agent security** – judging whether an action is “secure” depends on the actor, object, type, and contextual norms of the operation.

### Comparison with Static Policies

| Dimension | Traditional Static Policies | Conseca Dynamic Policies |
|-----------|-----------------------------|---------------------------|
| Policy generation | Human‑prewritten | LLM‑generated on‑the‑fly |
| Context awareness | Limited or none | Full context |
| Scalability | Low (cannot exhaust all scenarios) | High (self‑adaptive) |
| Human intervention | Pre‑configuration | In‑operation verification |
| False positives/negatives | Prevalent | Theoretically reducible |

---

## Research Setup

### Experimental Platforms and Models

Conseca builds on the team’s long‑term AI security research infrastructure:

- **Terrarium framework**: A dedicated experimental framework for security and privacy research in multi‑agent systems, supporting fine‑grained control and observation of multi‑agent interactions.
- **Base models**: The study involves multiple mainstream LLMs as agent backends, including Gemini, GPT, and Mistral.
- **Attack testing**: Evaluations include typical attack vectors such as context‑hijacking attacks and prompt injection attacks.

### Research Team Background

The research is a joint effort by the **UMass Amherst AI Security Lab** and **Google Research**. The author, Eugene Bagdasarian, is simultaneously an Assistant Professor at UMass and a part‑time Senior Research Scientist at Google, with a research focus on security and privacy attack vectors for AI systems in real‑world deployment. The team’s work has received support from the **Schmidt Sciences AI Safety Grant**.

---

## Comprehensive Analysis

### Why “Context” is the Key

The core thesis of the paper reveals a simple yet often overlooked truth: **security is not an inherent property of an action, but a property of the relationship between the action and its context**. Deleting an email is neither secure nor insecure in itself – it depends on who deletes it, what is deleted, why, and under what circumstances.

Current security defences for LLM agents often focus on paradigms like “instruction‑data separation”. However, as the authors’ subsequent research reveals, **the essence of prompt injection is not “data hiding instructions”, but “context being manipulated”**. When an email says “this request has been approved by the department head”, the issue is not whether this sentence is an instruction or data, but **whether this statement is true in the context**. This implies that any attempt to defend against attacks by simply separating instructions and data essentially evades the core problem.

### The “Impossible Triangle” of Static Policies

Conseca actually uncovers a deeper issue: in the era of general‑purpose AI agents, traditional security policies face an **“impossible triangle”** – **security, usability, and manageability** cannot all be achieved simultaneously:
- To ensure **security**, policies need to be strict.
- To ensure **usability**, policies need to be flexible enough to accommodate various legitimate scenarios.
- To ensure **manageability**, policies need to be maintained and understood by humans.

Traditional static policies constantly compromise among these three – being strict hurts usability, being flexible hurts security, and as the number of scenarios grows exponentially, manageability becomes impossible. Conseca breaks this deadlock through **dynamic generation + human verification**: let the LLM bear the heavy burden of “generation” while letting humans handle the critical gate of “verification”.

### Limitations and Challenges

Of course, Conseca faces many challenges. As later studies point out, when using LLMs to generate adaptive security policies, **the LLM itself may misunderstand the task requirements**, leading to overly strict policies (blocking legitimate actions) or overly permissive policies (allowing insecure actions). Moreover, some research more radically argues that **AI agents may never be able to fully defend against prompt injection attacks** – because an attacker can always construct a context where the blocked information flow appears legitimate. These challenges precisely highlight the importance of the research direction opened by Conseca, and also indicate that this is a long road requiring continuous exploration.

---

## Practical Applications

### Recommendations for AI System Developers

1. **Re‑think security architecture**: When designing security for AI agents, do not rely solely on predefined permission lists or rule engines. Instead, consider introducing context‑aware dynamic policy generation mechanisms.

2. **Adopt Conseca’s design philosophy**: Even if a full implementation of Conseca is not immediately possible, improvements can be made in the following aspects:
   - Incorporate more contextual signals (user intent, task goal, operation history, etc.) into security decisions.
   - Design human‑understandable policy representations to facilitate auditing and debugging.
   - Establish verification mechanisms for policy generation to prevent security holes caused by LLM misjudgments.

3. **Pay attention to multi‑agent scenarios**: As multi‑agent systems become more prevalent, security boundaries will expand from “agent‑environment” to “agent‑agent” interactions, requiring more sophisticated context‑aware security mechanisms.

### Implications for Researchers

1. **Contextual security is an emerging and important research direction**: From Conseca to subsequent prompt injection studies, context is becoming a core paradigm in AI security research.

2. **Value of interdisciplinary integration**: Conseca brings the privacy theory of “Contextual Integrity” into the security domain, demonstrating the innovative potential of cross‑disciplinary theoretical transfer.

3. **Balancing “defence” and “usability”**: No security mechanism should come at the cost of the agent’s core functionality – finding the balance between security and utility will be a central challenge for future research.

---

## References

- Original paper: Tsai, L., & Bagdasarian, E. (2025). Contextual Agent Security: A Policy for Every Purpose. *HotOS '25*. https://arxiv.org/abs/2501.17070
- AirGapAgent: Bagdasarian, E., et al. (2024). AirGapAgent: Protecting Privacy‑Conscious Conversational Agents. *CCS '24*. https://arxiv.org/abs/2405.05175
- Terrarium framework: https://www.arxiv.org/abs/2510.14312
- Author homepage: https://people.cs.umass.edu/~eugene/
