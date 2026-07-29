# AgentVigil: Generic Black-Box Red-teaming for Indirect Prompt Injection against LLM Agents

## Paper Highlights
AgentVigil is **the first generic framework for automated discovery of indirect prompt injection vulnerabilities in black-box LLM agents**. Drawing inspiration from traditional software fuzzing, it combines Monte Carlo Tree Search (MCTS)–driven seed selection with an adaptive scoring strategy. On the AgentDojo and VWA‑adv benchmarks, it achieves **71% and 70% attack success rates**, respectively – nearly doubling the performance of baseline attacks – while demonstrating strong cross‑task and cross‑model transferability.

---

## Core Research Contributions

### Problem Definition
LLM agents, with their powerful planning and reasoning capabilities, can invoke external tools and interact with complex environments. However, this introduces a critical security risk: **indirect prompt injection**. An attacker injects malicious instructions into external data sources that the agent retrieves (e.g., web comments, emails, calendar events). When the agent processes these data, it may be tricked into executing the attacker’s tasks instead of the user’s original goals.

Systematically evaluating agent systems against indirect prompt injection faces three major challenges:
1. **Black‑box nature** – real‑world agents are typically closed‑source, with no access to internal LLMs or architectures.
2. **Task diversity** – agents handle numerous dynamic and heterogeneous user tasks.
3. **Architectural complexity** – agents comprise multiple interconnected components, tools, and services.

Existing approaches either rely on manual attack construction or are designed for specific agent types, lacking general applicability.

### Innovative Approach
AgentVigil innovatively transfers the **fuzzing** paradigm from traditional software security to the LLM agent domain, with three key design adaptations:

- **High‑quality initial seed corpus** – compiled from heuristic rules, online resources, and prior prompt‑injection research, covering diverse strategies such as role‑play, delimiter attacks, and prompt obfuscation.
- **Adaptive seed scoring** – a hybrid evaluation mechanism that combines Attack Success Rate (ASR) with coverage‑oriented rewards, encouraging both immediate attack effectiveness and the ability to cover previously failed tasks.
- **MCTS‑based seed selection** – using the UCB1 algorithm to dynamically identify and prioritise valuable seeds, balancing exploration and exploitation.

Additionally, five mutation operators are implemented – **Shorten**, **Expand**, **Rephrase**, **Crossover**, and **GenerateSimilar** – executed by a helper LLM (e.g., Llama‑3‑8B or GPT‑4o‑mini).

### Research Findings
- **Benchmark performance**: On AgentDojo (personal assistant) and VWA‑adv (web agent), against o3‑mini and GPT‑4o based agents, AgentVigil achieves **71% and 70% ASR**, respectively. In contrast, manually crafted baseline attacks on AgentDojo reach only 38% – nearly doubled by AgentVigil.
- **Transferability**: Generated adversarial prompts transfer to unseen tasks with **65% and 59%** success on o3‑mini and GPT‑4o; against Gemini‑2‑flash‑exp (not seen during fuzzing), they still achieve **67%** ASR.
- **Defence evasion**: AgentVigil effectively bypasses multiple deployed defences on AgentDojo, including ProtectAI’s `pi_detector`, instruction repetition, and delimiter‑based filters.
- **Ablation studies**: Removing either the high‑quality initial corpus or the adaptive scoring + MCTS selection significantly reduces coverage growth (see Figure 3 in the paper).
- **Real‑world validation**: The framework successfully misled a real agent to navigate to arbitrary URLs (including malicious or download links), confirming practical applicability.

### Potential for Practical Deployment
AgentVigil serves as a **red‑teaming tool** of high practical value. Enterprises and security teams can use it to automatically assess the vulnerability of their LLM agents to indirect prompt injection **before deployment**, without requiring internal access – which perfectly fits commercial black‑box agents (e.g., OpenAI’s Operator, Anthropic’s Computer Use). Its modular design also allows adaptation to various agent types, from personal assistants to web automation agents.

---

## Technical Details

### System Architecture
AgentVigil’s overall workflow is illustrated in Figure 2 of the paper and consists of the following core modules:

```
Initial Seed Corpus
       │
       ▼
┌──────────────────────────────────────────────┐
│           Iterative Fuzzing Loop             │
│  ┌──────────────┐      ┌──────────────────┐  │
│  │ MCTS Seed    │ ──→  │ Mutation Engine  │  │
│  │ Selector     │      │ (5 operators)    │  │
│  └──────────────┘      └──────────────────┘  │
│        ↑                         │            │
│  ┌──────────────┐      ┌────────▼─────────┐  │
│  │ Seed Store   │ ←──  │ Scorer (ASR +    │  │
│  │              │      │ Coverage Bonus)   │  │
│  └──────────────┘      └──────────────────┘  │
└──────────────────────────────────────────────┘
       │
       ▼
Target LLM Agent System
```

### Seed Scoring Formula
Each seed receives a final score as a weighted sum of attack success rate and coverage bonus:

$$\text{Score}(s) = \alpha \cdot \text{ASR}(s) + \beta \cdot \text{CoverageBonus}(s)$$

where ASR is the ratio of successfully attacked tasks to total tasks, and CoverageBonus rewards seeds that succeed on previously failed tasks.

### Seed Selection: UCB1 Algorithm
The MCTS selector uses UCB1 to balance exploration and exploitation:

$$\text{UCB}(node) = \frac{Q(node)}{N(node)} + c \cdot \sqrt{\frac{\ln N(parent)}{N(node)}}$$

The first term represents exploitation (empirical performance of the node), and the second is the exploration bonus – ensuring that branches with fewer visits but high potential are adequately explored.

### Mutation Operators

| Operator | Function |
|----------|----------|
| Shorten | Compresses the seed for conciseness |
| Expand | Adds extra contextual information |
| Rephrase | Introduces linguistic variation while preserving semantics |
| Crossover | Combines elements from two parent seeds |
| GenerateSimilar | Creates new seeds with similar style but different content |

These mutations are performed by a helper LLM (e.g., Llama‑3‑8B or GPT‑4o‑mini), requiring only moderate model capability.

---

## Experimental Setup

### Threat Model
- **Black‑box setting**: The attacker has no access to the underlying LLM or agent architecture.
- **User assumption**: Users are benign and do not collude with the attacker.
- **Attacker capability**: The attacker can interact with the agent like any legitimate user and manipulate external data sources (e.g., product reviews, calendar entries) to inject malicious content.
- **Feedback**: The attacker only receives a binary signal (success/failure) via environment checks.

### Configuration Details
- **Benchmarks**: AgentDojo (personal assistant, with 142 fuzzing tasks and 173 test tasks) and VWA‑adv (web agent).
- **Target models**: o3‑mini (primary fuzzing target), GPT‑4o, GPT‑4o‑mini, Claude‑3.5‑Sonnet, Gemini‑2‑flash‑exp.
- **Helper model**: GPT‑4o‑mini used as the mutation LLM.
- **Iterations**: 3 mutated seeds per round, for 10 fuzzing rounds.
- **Metrics**: Attack Success Rate (ASR) and task coverage.

### Hardware / Software Requirements
Although the paper does not specify exact hardware, the following are inferred:
- Environment capable of running target LLM agents (API access or local deployment).
- A helper LLM (GPT‑4o‑mini or Llama‑3‑8B) for mutation.
- Benchmark evaluation frameworks (AgentDojo, VWA‑adv) installed and configured.

---

## In‑Depth Analysis

### Methodological Contributions
AgentVigil is the **first work to systematically adapt fuzzing to indirect prompt injection for LLM agents**. Unlike direct prompt injection (e.g., GPTFuzzer), where the attacker fully controls the input, indirect injection only allows influencing external content – greatly limiting attack capabilities. AgentVigil overcomes this via carefully designed initial seeds, MCTS‑guided search, and adaptive scoring, achieving efficient vulnerability discovery under constrained conditions.

Key differences from GPTFuzzer:
- GPTFuzzer targets **direct** prompt injection (jailbreak) – full input control.
- AgentVigil targets **indirect** injection – only external data influence.
- GPTFuzzer is single‑turn; AgentVigil handles multi‑step agents.

### Limitations and Notable Observations
1. **Significant model robustness variation**: Claude‑3.5‑Sonnet exhibits exceptional robustness – neither baseline nor AgentVigil‑generated prompts are effective. This suggests fundamental differences across model families that merit further investigation.
2. **Fragility of defences**: AgentVigil successfully bypasses multiple defences (dedicated detectors, instruction repetition, delimiters), indicating that current defensive measures remain inadequate against automated, adaptive attacks.
3. **Practical value of black‑box setting**: Commercial agents are typically black‑box; AgentVigil’s independence from internal access makes it highly practical.
4. **Dual nature of transferability**: The generated adversarial prompts transfer across tasks and models – this underscores the generality of the attacks, but also means that a single compromised agent could lead to reusable attack patterns for other systems.

### Broader Implications for Security
This work reveals a deeper issue: **context pollution** in LLM agents. In traditional software security, we worry about code injection; in the LLM agent era, prompt injection has become the new injection attack paradigm. AgentVigil demonstrates that such attacks are not occasional expert‑crafted exploits – they can be discovered and weaponized at scale through systematic fuzzing. This poses a severe challenge to the secure design of the entire LLM agent ecosystem.

---

## Practical Applications

### 1. Enterprise Security Assessment
Before deploying LLM‑based agents (e.g., customer service bots, automated office assistants, web automation), enterprises can use AgentVigil for **automated red‑teaming**:
- Integrate the agent into the framework.
- Define business‑relevant user tasks and attack goals.
- Run the fuzzing cycle to identify vulnerabilities.
- Harden the system accordingly (e.g., stricter external data filtering, tool‑call verification).

### 2. Security Product Development
Security vendors can incorporate AgentVigil’s methodology into LLM security assessment platforms, offering **black‑box security auditing** as a service. The framework’s independence from internal access makes it well‑suited for SaaS‑based delivery.

### 3. Defence Validation
When developing new defence mechanisms, AgentVigil can serve as an **adversarial evaluation tool** to test robustness against diverse attack strategies. The paper’s findings that existing defences perform poorly under AgentVigil highlight the urgent need for stronger defences.

### 4. Ethical and Responsible Use
AgentVigil is fundamentally an **attack tool**. Its use must adhere to responsible disclosure principles:
- Test only on your own systems with proper authorisation.
- Disclose discovered vulnerabilities through appropriate channels.
- Do not use generated adversarial prompts for unauthorised real‑world attacks.

---

## References
- Original Paper: [AgentVigil: Generic Black-Box Red-teaming for Indirect Prompt Injection against LLM Agents](https://arxiv.org/abs/2505.05849) (arXiv:2505.05849v4, Jun 2025)
- Full PDF: [https://arxiv.org/pdf/2505.05849](https://arxiv.org/pdf/2505.05849)
