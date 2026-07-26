# Agent Security Bench (ASB): Formalizing and Benchmarking Attacks and Defenses in LLM‑based Agents

## Paper Highlights

LLM-based agents, with their ability to invoke external tools and leverage memory for complex tasks, introduce a broad new attack surface—yet no systematic evaluation framework has existed. ASB fills this gap by providing a unified benchmark covering 10 scenarios, 10 agent types, 400+ tools, 27 attack/defense methods, and 7 evaluation metrics. Through large‑scale experiments on 13 LLM backbones with nearly 90,000 test cases, ASB reveals severe vulnerabilities across system prompts, user inputs, tool calls, and memory retrieval—with the highest average attack success rate reaching **84.30%**—while existing defenses show only limited effectiveness.

---

## Core Research Content

### Problem Definition

The rise of LLM‑based agents moves AI from “chatting” to “acting”—they not only answer questions but also search the web, read/write files, operate databases, and control APIs. This action capability opens up entirely new attack vectors. A traditional LLM, if jailbroken, might produce harmful text; a compromised agent, however, could delete your files, transfer funds, or leak private data.

Previous security evaluations have been fragmented—some focus only on prompt injection in conversational settings, others study only a single type of agent. There is no unified, reproducible benchmark. ASB addresses the core question: **How vulnerable are LLM‑based agents?** From which angles can they be attacked, and do existing defenses actually work?

### Innovative Approach

ASB contributes on three levels:

**1. Formalisation of the attack/defence space.** It systematically maps the critical stages of agent operation—system prompt (role and behaviour definition), user instruction parsing, memory retrieval, planning/reasoning, and tool execution—and designs corresponding attack vectors for each.

**2. Large‑scale benchmark construction.** This is not a toy demo. It covers 10 realistic scenarios (e‑commerce, autonomous driving, finance, academic consulting, mental health, investment, legal, etc.), contains 400+ tools, and supports 13 LLM backbones. Attack types include 10 prompt injection variants, memory poisoning, and the paper’s original **Plan‑of‑Thought (PoT) backdoor** and **mixed attacks**.

**3. Utility‑aware evaluation metrics.** Looking at attack success rate alone is misleading—an agent that rejects everything is “safe” but useless. ASB introduces new metrics that assess how well an agent maintains its primary task performance while resisting attacks.

### Research Results

The experimental data are alarming. Under comprehensive testing, various attacks achieve high success rates across all stages of agent operation, with **the highest average ASR reaching 84.30%**. This means that, under certain attack configurations, more than 8 out of 10 attempts successfully force the agent to execute the attacker’s desired malicious behaviour.

Even more concerning, the 11 existing defense methods tested show generally poor performance. This exposes an uncomfortable reality: on the agent security battlefield, attackers have already run far ahead, while defenders are still struggling to keep pace.

### Real‑world Deployment Potential

ASB is not just an academic paper—it is a **directly runnable codebase** built on AIOS (Agent Intelligence Operating System), with YAML configuration files for flexible attack parameters and LLM backends.

Practical applications:

- **Enterprise security teams** can use ASB to red‑team their own agent systems before deployment.
- **Agent developers** can integrate ASB into CI/CD pipelines for continuous security regression testing.
- **Cloud providers and model vendors** can benchmark different models and defense strategies against each other, guiding selection and optimisation.

---

## Technical Details

### Attack Taxonomy

ASB classifies attacks into four major categories:

**1. Direct Prompt Injection (DPI)**  
An attacker embeds malicious instructions in user input, attempting to override or bypass the safety constraints defined in the system prompt. This is the most straightforward and common attack.

**2. Observation Prompt Injection (OPI)**  
The attack does not come from the user directly, but from external information observed during agent execution—such as web page content, API responses, file contents, etc. This “indirect injection” is harder to defend against because the attacker does not need to interact with the agent directly.

**3. Memory Poisoning**  
Agents typically maintain a memory bank to store historical interactions and retrieved information. An attacker pollutes the memory content, so that future retrievals return tampered information, influencing the agent’s decision‑making.

**4. Plan‑of‑Thought (PoT) Backdoor Attack**  
This is an original attack proposed by the paper. Unlike traditional Chain‑of‑Thought (CoT) backdoors that trigger during reasoning, PoT attacks act on the agent’s **planning phase**. Through specific trigger patterns, the agent is induced to produce preset malicious actions when formulating its plan.

### Evaluation Metrics

ASB designs 7‑8 evaluation metrics, including:

- **Attack Success Rate (ASR)** – the proportion of successful attacks.
- **Task Completion Rate** – the agent’s ability to still complete its original task under attack.
- **Safety‑Utility Balance** – a signature metric of ASB that measures the trade‑off between security and usefulness.

### Experiment Scale

- **13 LLM backbones**: GPT‑4o, GPT‑4o‑mini, Claude‑3, Gemini‑2.0‑flash, Llama‑3.3‑70B, and others.
- **Nearly 90,000 test cases**.
- **10 realistic scenarios** and corresponding **10 agent types**.

---

## Experimental Setup

### Architecture

ASB is built on **AIOS** (Agent Intelligence Operating System), which provides a standardised runtime environment, enabling uniform attack injection, logging, and evaluation.

The standard execution flow for each agent in ASB:
1. System prompt defines role and behaviour constraints.
2. User instruction is received.
3. Relevant information is retrieved from memory.
4. Planning is performed based on retrieval results and context.
5. External tools are invoked to execute actions.

ASB’s attack injection points cover all the above stages.

### Hardware & Software Requirements

**Environment:**
- Python 3.11
- Conda environment management
- Dependencies installed via `pip install -r requirements.txt`

**LLM backends:**
- **Closed‑source models**: via API (e.g., OpenAI GPT‑4o, Anthropic Claude)
- **Open‑source models**: via local Ollama deployment (CPU‑only mode supported, no CUDA required)

**Execution examples:**
```bash
# Direct Prompt Injection
python scripts/agent_attack.py --cfg_path config/DPI.yml

# Observation Prompt Injection
python scripts/agent_attack.py --cfg_path config/OPI.yml

# Memory Poisoning
python scripts/agent_attack.py --cfg_path config/MP.yml

# Mixed Attack
python scripts/agent_attack.py --cfg_path config/mixed.yml

# PoT Backdoor Attack
python scripts/agent_attack_pot.py
```

**Configuration flexibility:**  
The YAML files allow free combination of different LLM backbones, attack methods, scenarios, and agent configurations. For example, testing GPT‑4o and LLaMA3.1‑70B simultaneously is as simple as listing both models in the YAML.

---

## Comprehensive Analysis

### Why 84.30% ASR is a Wake‑up Call

The 84.30% figure reflects a structural problem, not a momentary lapse of any particular model.

The vulnerability of LLM‑based agents is rooted in their architectural design. Traditional software has clear security boundaries—inputs are validated, permissions are checked, operations are audited. But an agent is essentially a system that **translates natural language into actions**, and natural language is inherently ambiguous, context‑sensitive, and difficult to verify formally.

When an attacker says “Please ignore all previous instructions,” it is not a traditional “injection attack”—no buffer overflow, no SQL syntax error, no XSS script. It is simply **human language**. And precisely because it is human language, the LLM “understands” and “obeys” it. The smarter the agent, the better it understands such instructions—and the more easily it is manipulated by them.

This is the core paradox of agent security: **intelligence itself is the vulnerability**.

### The Deeper Implications of PoT Backdoor Attacks

The introduction of PoT backdoor attacks deserves special attention. Traditional prompt injections are essentially “instruction overriding”—new instructions overwrite old ones. But PoT attacks operate at the **planning level**: the agent is not executing a “wrong” instruction; it is “correctly” planning a “wrong” course of action.

This makes the attack much harder to detect, because from the logs, everything appears legitimate—the agent is simply following its planning logic, but that logic has been subtly twisted.

### Why Defenses Fail

The paper shows that 11 existing defenses have limited effect. This is unsurprising. Most current defenses are essentially content filters—they try to detect malicious keywords or attempts to bypass system prompts. But against sophisticated attacks (especially planning‑level ones like PoT), simple filtering is like trying to stop bullets with a fishing net.

True defense may require a fundamental re‑thinking of agent architecture: **how can we make an agent “understand” instructions while simultaneously “not trusting” them?** How can we preserve intelligence while erecting an unbreakable safety boundary? These questions remain unanswered.

---

## Practical Guidance

### If You Are an Agent Developer

1. **Red‑team before deployment** – Use ASB to thoroughly evaluate your agent’s security before going live.
2. **Establish continuous security regression** – Integrate ASB into your CI/CD pipeline to automatically run security tests with every update.
3. **Watch for indirect injection** – If your agent reads web pages, emails, or documents, OPI attacks are your biggest threat.

### If You Are a Security Engineer

1. **Don’t blindly trust existing defenses** – The paper demonstrates that most off‑the‑shelf solutions perform poorly under ASB. Validate any solution with ASB before purchase.
2. **Think at the architectural level** – Prompt filtering and input validation are necessary but far from sufficient. Build security into the planning, execution, and memory stages.
3. **Watch multi‑agent scenarios** – ASB currently focuses on single agents, but multi‑agent collaboration introduces even more complex risks—a compromised agent can become a springboard to attack others.

### If You Are a Researcher

1. **ASB is a ready‑made experimental platform** – Code, scenarios, tools, and metrics are all open‑sourced. Use it to develop new attacks or defences.
2. **Safety‑utility balance is a rich direction** – ASB’s emphasis on this trade‑off is valuable, but research in this area is still nascent.
3. **Focus on defence, not just discovery** – With ASR already at 84.30%, the community has done well at finding holes; the real challenge is fixing them.

---

## References

- Original paper: Zhang, H., Huang, J., Mei, K., et al. "Agent Security Bench (ASB): Formalizing and Benchmarking Attacks and Defenses in LLM‑based Agents." *ICLR 2025*. https://arxiv.org/abs/2410.02644
- Official code repository: https://github.com/agiresearch/ASB
- Project website: https://luckfort.github.io/ASBench/
