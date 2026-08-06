
# Here Comes The AI Worm: A Technical Analysis of the Morris-II Worm

This document provides a comprehensive, in‑depth analysis of the research paper  
**“Here Comes The AI Worm: Unleashing Zero‑click Worms that Target GenAI‑Powered Applications”**  
by Stav Cohen, Ron Bitton, and Ben Nassi (arXiv:2403.02817, 2024).  

It is intended for security researchers, AI practitioners, and anyone interested in the emerging threat landscape of generative AI ecosystems.

---

## Key Takeaways

- **First‑of‑its‑kind malware** – The authors present *Morris II*, the first known worm that propagates autonomously through generative AI (GenAI) applications using adversarially crafted prompts.
- **Two propagation vectors** – The worm spreads via **RAG‑based databases** (active retrieval) and **application workflow** (automatic processing of incoming data), achieving zero‑click infection.
- **Effective defense** – The proposed *Virtual Donkey* shield detects the worm with **perfect true‑positive rate (1.0)** and **extremely low false‑positive rate (0.015)**, and is robust against out‑of‑distribution variants.
- **Real‑world implications** – As GenAI integrates into operating systems, smartphones, and vehicles, this work highlights a critical new attack surface that demands immediate attention from the community.

---

## Core Research Content

### Problem Definition

With the rapid adoption of GenAI, many applications now incorporate large language models (LLMs) and form an interconnected ecosystem of semi‑/fully autonomous agents. Prior security research focused mainly on single‑model threats (e.g., prompt injection, membership inference, data leakage). However, an open question remained: **Can an attacker develop malware that propagates at the *ecosystem* level by exploiting GenAI components?** This paper answers that question affirmatively.

### Innovative Approach

The key innovation is the concept of **adversarial self‑replicating prompts** – inputs that cause the GenAI model to confuse *data* with *code*, analogous to SQL injection or buffer overflows in traditional security. The approach includes:

- **Replication mechanisms**:
  - *Direct replication* – the model outputs the input exactly as given.
  - *Conditional replication* – the prompt is embedded within a larger context, and the model outputs both a malicious payload and a copy of the prompt.
- **Propagation vectors**:
  - *RAG‑based* – the malicious prompt is stored in a retrieval‑augmented generation (RAG) database and later retrieved in subsequent responses.
  - *Workflow‑based* – the prompt manipulates the application’s execution flow so that the output is passed to other agents.
- **Zero‑click** – Because many GenAI apps automatically process incoming data (e.g., email attachments, images), the worm triggers without any user interaction.

### Research Outcomes

The authors evaluated Morris II in multiple dimensions:

- **Test scenarios** – Two use‑cases on a GenAI‑powered email assistant: *spam distribution* and *personal data exfiltration*.
- **Access modes** – Both black‑box and white‑box settings.
- **Input types** – Text and image carriers.
- **Target models** – Gemini Pro, ChatGPT‑4.0, and LLaVA.

For defense, they propose **Virtual Donkey**, a lightweight shield that detects and blocks Morris II propagation. It achieves:
- **True‑positive rate:** 1.0 (perfect detection)
- **False‑positive rate:** 0.015 (extremely low)
- **Robustness** – effective against unseen, out‑of‑distribution variants.

### Practical Applicability

The findings carry significant real‑world weight:

- **Security alert** – They reveal a new threat vector that will become more severe as GenAI is embedded into critical systems.
- **Defense tooling** – Virtual Donkey serves as a proof‑of‑concept for building practical, deployable protections.
- **Industry standards** – The research urges a rethinking of RAG architecture, automatic response mechanisms, and overall GenAI system design.

---

## Technical Details

### Formal Definition of Adversarial Self‑Replicating Prompts

Let \( G \) be a GenAI model, input \( x \), output \( G(x) \). Two forms are defined:

**Direct replication:**  
\[
G(x) \rightarrow x
\]  
The input and output are identical. For example, an image containing an embedded prompt causes the model to reproduce that prompt.

**Conditional replication:**  
\[
G(w \parallel x \parallel y) \rightarrow \text{payload} \parallel x
\]  
The prompt \( x \) is part of a larger context; the output includes a malicious payload along with a copy of \( x \).

### Workflow of the RAG‑based Worm (Email Assistant Scenario)

1. **Initial infection** – Attacker \( u_1 \) sends an email \( e_1 \) containing the adversarial self‑replicating prompt to target user \( u_2 \).
2. **Context retrieval** – The client of \( u_2 \) retrieves the top‑\( k \) relevant historical communications from the RAG database as context.
3. **Automatic reasoning** – The client queries the GenAI service to generate an automatic reply based on that context.
4. **Reply propagation** – The generated output is sent back to \( u_1 \) as a reply.
5. **Database poisoning** – The new communication is stored into the RAG database – the adversarial prompt is now persistently written and ready for future retrieval.

This cycle enables self‑replication and persistent infection across the ecosystem.

### Prompt Construction Methods

Attackers can craft effective prompts via:
- **Fuzzing** – Black‑box queries to the GenAI model to discover inputs that trigger replication.
- **Leveraging known jailbreaks** – Many jailbreak prompts are publicly shared (blogs, forums) and can be adapted.

---

## Experimental Setup

### Environment

- **Target application** – GenAI‑powered email assistant with automatic reply capability.
- **RAG configuration** – Active database that automatically stores new emails.
- **GenAI models** – Gemini Pro, ChatGPT‑4.0, LLaVA.

### Attacker Capabilities

The attacker is assumed to be able to:
1. Craft a message containing the adversarial self‑replicating prompt.
2. Have that message stored in the victim’s RAG database (automatic storage by the active RAG).
3. Have it retrieved in subsequent responses.
4. Trigger replication during GenAI inference.
5. Simultaneously execute a predefined malicious payload.

### Defense Configuration

Virtual Donkey is designed to detect and block worm propagation with minimal latency, without degrading user experience. Evaluation metrics include true‑positive rate, false‑positive rate, and robustness against unseen variants.

---

## Comprehensive Analysis

### Academic Significance

- **Bridging a gap** – Prior work concentrated on single‑model risks; this paper elevates the threat to the *ecosystem* level, opening a new research direction.
- **Methodological innovation** – It successfully transplants the classic *worm* concept into the AI domain, showing that the data‑code confusion vulnerability reappears in GenAI.
- **Historical nod** – The name *Morris II* pays tribute to the 1988 Morris worm, signalling that the AI era’s equivalent has now arrived.

### Technical Limitations

- **Environment‑dependent** – Effectiveness relies on RAG architectures and automatic processing; not all GenAI apps are susceptible.
- **Model variability** – Sensitivity differs across models; new models may require re‑evaluation.
- **Passive defense** – Virtual Donkey is a detection‑based shield; it may lag behind rapidly evolving adversarial prompts.

### Real‑world Threat Assessment

- **Low barrier** – Attackers need not be AI experts; public jailbreaks lower the entry threshold.
- **High impact** – As GenAI integrates into OS, smartphones, and vehicles, potential payloads expand from spam and data theft to ransomware and remote code execution.
- **Hard to detect** – Traditional signature‑based security is ineffective against prompt‑based attacks.

---

## Practical Recommendations

### For GenAI Application Developers

- **Rethink RAG security** – Implement strict input filtering and output sanitisation, especially for active databases.
- **Limit automatic processing** – Apply sandboxing and pre‑processing to automatically handled inputs (e.g., images, email bodies).
- **Deploy dedicated shields** – Adopt lightweight detection modules inspired by Virtual Donkey.

### For AI Platform Providers

- **Enhance model robustness** – Include adversarial training in fine‑tuning pipelines.
- **Share threat intelligence** – Establish cross‑platform information sharing on adversarial prompts.
- **Offer security configurations** – Provide granular controls to disable auto‑reasoning or restrict context retrieval.

### For End Users

- **Grant permissions cautiously** – Be mindful when granting access to sensitive data (email, contacts).
- **Watch for anomalies** – Report unusual AI responses to providers.
- **Keep software updated** – Apply security patches as soon as they are available.



## References

- Original paper: Cohen, S., Bitton, R., & Nassi, B. (2024). *Here Comes The AI Worm: Unleashing Zero‑click Worms that Target GenAI‑Powered Applications*. arXiv:2403.02817.  
  [https://arxiv.org/abs/2403.02817](https://arxiv.org/abs/2403.02817)
- Companion code (ComPromptMized): [https://github.com/genaforvena/ComPromptMized](https://github.com/genaforvena/ComPromptMized)
```
