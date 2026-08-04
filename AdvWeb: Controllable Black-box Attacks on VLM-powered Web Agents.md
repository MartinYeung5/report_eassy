# AdvWeb: Controllable Black-box Attacks on VLM-powered Web Agents

## Paper Highlights

The rapid advancement of vision-language models (VLMs) has given rise to general-purpose Web Agents capable of autonomously completing tasks on real‑world websites. However, the security of these agents against adversarial attacks remains largely unexplored. AdvWeb proposes a novel black‑box attack framework that trains an adversarial prompter model to generate and inject visually imperceptible adversarial strings into web pages, misleading VLM‑powered Web Agents into executing preset harmful actions—such as inappropriate stock purchases or erroneous bank transactions.

---

## Core Research Contributions

### Problem Definition

With the proliferation of LLMs and VLMs, general‑purpose Web Agents can now interact autonomously with real websites and perform tasks across high‑stakes domains like finance, healthcare, and e‑commerce. Existing adversarial attacks against Web Agents have notable shortcomings: they are either too costly (requiring manual prompt engineering), focus on a single attack scenario with limited flexibility, or lack transferability under black‑box settings. Thus, the core challenge is: **How can we efficiently, flexibly, and stealthily attack VLM‑driven Web Agents with only black‑box access?**

### Innovative Approach

AdvWeb introduces several key innovations:

- **Two‑Stage Training Paradigm** – A reinforcement‑learning‑inspired two‑stage pipeline that leverages black‑box feedback from the victim agent to optimise adversarial strings. Specifically, it employs Direct Preference Optimisation (DPO) using both successful and failed attack strings to train the adversarial prompter.

- **Stealthiness and Controllability** – Unlike previous methods, AdvWeb offers:
  - *Stealthiness*: The visual appearance of the attacked web page remains unchanged, making the manipulation nearly imperceptible to users.
  - *Controllability*: Attackers can modify specific sub‑strings within the generated adversarial prompt to seamlessly switch attack targets (e.g., changing the target company for stock purchases) without retraining the model.

- **Black‑Box Setting** – AdvWeb requires only black‑box access to the Web Agent, meaning no knowledge of its internal architecture or parameters is needed—a far more realistic threat model.

### Research Results

Extensive experiments demonstrate that AdvWeb achieves **high attack success rates** against state‑of‑the‑art VLM agents (e.g., GPT‑4V‑based) across diverse web tasks under black‑box conditions. These results expose critical security vulnerabilities in current LLM/VLM‑based agents and underscore the urgent need for more robust Web Agents and effective defensive measures.

### Real‑World Applicability

- **On the attack side**: AdvWeb can be used by red teams to conduct security assessments before Web Agents are deployed, helping to uncover potential weaknesses.
- **On the defence side**: The vulnerabilities revealed by this work provide clear directions for developing detection and mitigation strategies against such visually imperceptible adversarial injections.

---

## Technical Details

### Core Architecture

AdvWeb consists of three key components:

1. **Target Web Agent** – A VLM‑based (e.g., GPT‑4V) general‑purpose agent that understands web page content and executes actions.
2. **Adversarial Prompter** – A trained model that generates adversarial strings according to the attack objective. It is optimised via DPO using black‑box feedback from the target agent.
3. **Injection Mechanism** – Inserts the generated adversarial strings into specific locations on the web page (e.g., invisible elements or page text) while preserving the page’s visual appearance.

### DPO Training Mechanism

AdvWeb uses Direct Preference Optimisation instead of traditional reinforcement learning, which avoids training a separate reward model. In this context:
- **Successful attack strings** – those that successfully mislead the agent into performing the target action – serve as positive samples.
- **Failed attack strings** – those that do not achieve the desired effect – serve as negative samples.

Through DPO, the adversarial prompter learns to generate strings that are more likely to succeed against the victim agent.

### Controllable Editing of Adversarial Strings

A notable feature is the ability to switch attack targets on the fly. The generated adversarial string contains editable “placeholder” sub‑strings. For example:

> Original: `"... purchase stocks of [COMPANY] ..."`  
> Modified: `"... purchase stocks of Tesla ..."`  
> Re‑modified: `"... purchase stocks of Apple ..."`

This design allows attackers to change targets without retraining, greatly improving efficiency.

---

## Experimental Setup

### Configuration Overview

| Aspect | Details |
|--------|---------|
| **Paper length** | 15 pages |
| **Primary fields** | Cryptography and Security (cs.CR); Computation and Language (cs.CL) |
| **Target agent** | SOTA VLM agents (e.g., GPT‑4V) |
| **Attack setting** | Black‑box (only input‑output observations, no internal access) |
| **Training algorithm** | Direct Preference Optimisation (DPO) |
| **Evaluation tasks** | Diverse real‑world web tasks (stock trading, banking operations, etc.) |

### Code and Data Availability

The authors have publicly released code and data at:
- **Project page**: https://ai-secure.github.io/AdvWeb/
- **Paper preprint**: https://arxiv.org/abs/2410.17401

### Hardware Requirements (Inferred)

Although not explicitly stated, the following can be reasonably inferred:
- **Training**: GPU resources are needed for DPO optimisation of the adversarial prompter (scale depends on the base model size).
- **Inference**: Running the trained prompter is relatively lightweight.
- **Agent queries**: API access to a VLM (e.g., GPT‑4V) is required for obtaining feedback.

---

## Comprehensive Analysis

### Academic Significance

AdvWeb makes important contributions to security research. First, it extends adversarial attacks from isolated LLM/VLMs to **agent systems** that interact with real‑world web pages—this expands the attack surface and amplifies potential harm. Second, the **black‑box assumption** is far more practical, as deployed Web Agents rarely expose their internals. Third, the use of **DPO** provides an efficient path for automated adversarial prompt generation, avoiding the overhead of reward‑model training.

### Technical Limitations

Several limitations deserve attention:
- **API dependency**: Training requires repeated calls to the target agent, which may be constrained by cost or rate limits.
- **Generalisation**: The evaluation focuses mainly on GPT‑4V; effectiveness against other VLMs (e.g., Qwen‑VL, LLaVA) needs further validation.
- **Evolving defences**: As defences improve, AdvWeb’s effectiveness may degrade, necessitating continuous iteration.

### Practical Implications

Perhaps the most alarming finding is that **attacks can occur without any user awareness**. Since the web page looks identical, users have no reason to suspect tampering; meanwhile, the Web Agent unknowingly executes harmful operations that could lead to actual financial losses. This “neither the user nor the agent notices” threat model poses a serious risk to high‑value applications such as financial transactions, medical record modifications, and e‑commerce.

### Comparison with Prior Work

| Dimension | Traditional Adversarial Attacks | AdvWeb |
|-----------|--------------------------------|--------|
| Target | LLM/VLM models | VLM‑driven **Web Agents** |
| Access | Often white‑box or grey‑box | **Pure black‑box** |
| Stealthiness | Obvious text perturbations | **Visual appearance unchanged** |
| Controllability | Fixed target | **Editable sub‑strings** |
| Automation | Manual prompt engineering | **DPO‑based automatic optimisation** |

---

## Practical Recommendations

### For Security Researchers

- **Red‑team testing**: Integrate AdvWeb into pre‑deployment security pipelines to simulate realistic attacks and uncover vulnerabilities early.
- **Defence development**: Investigate detection and mitigation strategies, such as:
  - Adversarial robustness checks on web page content.
  - Filtering layers that detect adversarial inputs before agent decision‑making.
  - Secondary confirmation mechanisms for critical actions (e.g., transactions, modifications).

### For Web Agent Developers

- **Input sanitisation**: Incorporate modules that detect and filter adversarial strings before agent processing.
- **Output monitoring**: Implement anomaly detection on agent actions to identify patterns indicative of adversarial influence.
- **Adversarial training**: Augment training data with adversarial examples to improve robustness against such attacks.

### For End Users

While technical defences are not directly accessible to end users, they can:
- Choose Web Agent services that demonstrate robust security evaluation practices.
- Remain vigilant for sensitive operations (financial transactions, personal data changes) and verify them through multiple channels when possible.

---

## References

- Original paper: https://arxiv.org/abs/2410.17401
- Project page: https://ai-secure.github.io/AdvWeb/
- Hugging Face paper page: https://huggingface.co/papers/2410.17401
