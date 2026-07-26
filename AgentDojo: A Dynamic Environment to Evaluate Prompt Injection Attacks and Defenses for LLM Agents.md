
# AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents

## Paper Highlights

AgentDojo is a dynamic evaluation framework proposed by ETH Zurich and Invariant Labs to systematically measure the adversarial robustness of LLM agents during tool-calling. It includes 97 realistic tasks and 629 security test cases across scenarios such as email management, online banking, and travel booking, providing the first scalable, standardized evaluation platform for prompt injection attack and defense research.

## Core Research Content

### Problem Definition

LLM agents solve complex tasks by combining textual reasoning with external tool calls. However, this design paradigm has a fundamental security flaw: LLMs process text directly and lack a formal way to distinguish between "instructions" and "data." Prompt injection attacks exploit this vulnerability—attackers embed malicious instructions into third‑party data (e.g., email bodies, web pages, API responses). When an agent retrieves such data via tools, the malicious instructions can hijack its behavior, forcing the agent to execute attacker‑specified operations.

Existing benchmarks are either limited to static scenarios or rely on LLMs to simulate environment states, failing to accurately reflect agent behavior in real dynamic environments. AgentDojo fills this gap.

### Innovative Approach

AgentDojo’s core innovations lie in several dimensions:

**1. Dynamic, Stateful Evaluation Environment**

Unlike traditional static test sets, AgentDojo is an extensible interactive environment. Agents must **dynamically call multiple tools** within this environment, and each tool invocation changes the environment state. This means that attacks and defenses must be evaluated in the context of full task execution, rather than judging isolated model outputs.

**2. Formal Utility and Security Metrics Based on Environment State**

AgentDojo does **not** rely on LLMs to simulate the environment or judge task completion. Instead, it computes **utility** (task completion) and **security** (attack success) through formal checks on the **environment state**. For example, to determine whether "an email has been sent" or "sensitive data has been leaked," it directly inspects the email records or file system—avoiding the bias and instability of using LLMs as judges.

**3. Orthogonal Design of Tasks, Attacks, and Defenses**

AgentDojo decouples three dimensions: user task, attack, and defense. The same set of user tasks can be combined with different attack strategies and defense mechanisms, allowing systematic evaluation of utility‑security trade‑offs across various combinations. This design lets researchers improve one dimension independently without redesigning the entire experiment.

**4. Extensible Architecture**

AgentDojo is designed as an **extensible framework** rather than a closed benchmark. Researchers can easily add new task suites, new attack paradigms, new defense mechanisms, and even new agent architectures, enabling it to evolve alongside the LLM agent field.

### Research Findings

The paper reveals several key insights through extensive experiments:

**Basic agent capabilities are insufficient.** Even **without any attacks**, state‑of‑the‑art LLM agents fail on many tasks. This indicates that the foundational task‑execution ability of current agents itself has significant room for improvement—security issues are just the tip of the iceberg.

**Existing attacks have limited effectiveness.** Current prompt injection attacks can **compromise some security properties, but not all**. This means no single attack is universally effective; different attack strategies perform very differently across scenarios.

**AgentDojo poses a dual challenge.** For attackers, achieving the attack goal while maintaining the agent’s normal execution of the user task is a highly complex optimization problem. For defenders, resisting attacks without significantly degrading task utility is equally challenging.

The paper also reports that the framework has been adopted by the **US and UK AISI (AI Safety Institutes)** to evaluate prompt injection vulnerabilities in Claude 3.5 Sonnet.

### Practical Deployment Feasibility

AgentDojo’s practicality has been validated at multiple levels. First, it won the **first prize in the ML Safety SafeBench Competition**, confirming its academic recognition in AI safety evaluation. Second, its adoption by **US and UK AISI** for real agent security assessments shows that the framework has moved from academic research to practical use.

For enterprises, AgentDojo can be used to:
- Assess security risks of agent systems before deployment
- Compare utility‑security trade‑offs of different defense strategies
- Continuously monitor agent system performance under adversarial conditions

## Technical Details

### Framework Architecture

The core workflow of AgentDojo is as follows:

1. **Task Definition**: Each task includes a user instruction, an initial environment state, and formal conditions (utility checks) to determine task completion.
2. **Security Test Definition**: Each security test includes an attacker goal and an injection endpoint.
3. **Agent Execution**: The agent calls tools step by step in the environment; each step may trigger an injection.
4. **Result Evaluation**: Utility and security scores are calculated by inspecting the final environment state.

### Task Suites

The first release of AgentDojo includes three task suites:

| Suite | Scenario | Number of Tasks |
|-------|----------|----------------|
| Workspace | Email client and file system management | Various office tasks |
| Banking | Online banking operations | Transfers, inquiries, etc. |
| Travel | Travel booking | Flight and hotel reservations |

### Attack Paradigms

AgentDojo implements several prompt injection attack strategies, including:
- **Direct injection**: Embedding malicious instructions directly in tool‑returned data
- **Indirect injection**: Passing malicious content through third‑party sources (e.g., web pages, documents)
- **Tool‑knowledge attacks**: The attacker knows the available tools and their parameters to craft more precise injections

### Defense Paradigms

The framework includes several built‑in defense mechanisms for comparative evaluation:
- **Prompt Enhancement**: Reinforcing user instructions in the system prompt to prioritize them
- **Detection Filter**: Using dedicated models to detect and filter injected content
- **Tool Filter**: Isolating function calls from the agent’s main planning component

### Code Examples

Below are command‑line examples for running benchmarks with AgentDojo:

```bash
# Run tasks 0 and 1 on the workspace suite using gpt-4o, with tool_filter defense and tool_knowledge attack
python -m agentdojo.scripts.benchmark \
  -s workspace \
  -ut user_task_0 -ut user_task_1 \
  --model gpt-4o-2024-05-13 \
  --defense tool_filter \
  --attack tool_knowledge

# Run on all suites and tasks
python -m agentdojo.scripts.benchmark \
  --model gpt-4o-2024-05-13 \
  --defense tool_filter \
  --attack tool_knowledge
```

Installation:

```bash
pip install agentdojo
# To use the prompt injection detector
pip install "agentdojo[transformers]"
```

## Experimental Setup

### Hardware and Software Requirements

AgentDojo is a **lightweight Python framework** whose core logic does not depend on specific hardware. Main requirements include:

- **Python environment**: Python 3.8+
- **LLM backends**: Supports OpenAI API (e.g., GPT-4o), Anthropic API (e.g., Claude), and locally deployed open‑source models (e.g., Llama)
- **Optional dependencies**: The `transformers` library is required if using the built‑in prompt injection detector

### Experimental Design

The experiments in the paper used the following setup:

1. **Agents**: Multiple SOTA LLMs were tested as reasoning engines for the agents.
2. **Task set**: 97 realistic tasks across the three scenario suites.
3. **Attack set**: 629 security test cases.
4. **Evaluation metrics**:
   - **Utility**: Whether the task was successfully completed (based on formal environment‑state checks).
   - **Security**: Whether the attack goal was achieved (also based on environment‑state checks).
5. **Baselines**: Various combinations of attack strategies × defense strategies were compared.

## Comprehensive Analysis

AgentDojo marks a significant shift in LLM agent security evaluation—from static Q&A to dynamic interaction. Several perspectives are worth deeper reflection:

**First, the fundamental conflict between instructions and data.** The problem AgentDojo exposes is rooted in a core property of LLMs: the inability to distinguish "this is an instruction" from "this is data" within the text input. This problem was long solved in traditional computing systems (code/data separation), but it re‑emerges in the LLM paradigm. AgentDojo’s value lies in forcing researchers to confront this fundamental contradiction rather than avoid it.

**Second, the reality of the utility‑security trade‑off.** The paper finds that even SOTA models fail in attack‑free scenarios, highlighting a deeper reality: **security defenses cannot come at the expense of basic capability**. If a defense causes the agent to fail 99% of tasks in benign settings, it is useless in practice. By evaluating both utility and security simultaneously, AgentDojo compels researchers to face this trade‑off.

**Third, the importance of dynamic benchmarks.** Prompt injection attacks and defenses are rapidly evolving. Static benchmarks quickly become obsolete—attackers will find ways to bypass them, and defenders will develop new protections. AgentDojo’s extensible design philosophy (rather than a closed test set) is the right direction to address this challenge.

**Fourth, the complexity of real‑world deployment.** Although AgentDojo’s scenarios (email, banking, travel) are simplified versions of real tasks, they already reveal the high complexity of agent security. In actual deployment, agents may face larger tool sets, more unpredictable data sources, and smarter adaptive attackers. AgentDojo provides a starting point, but it is far from the end.

## Practical Applications

Based on the analysis of AgentDojo, here are specific recommendations for researchers and practitioners:

### For Researchers

1. **Prioritize basic capability improvements**: Before pursuing security, ensure the agent achieves sufficiently high task completion in attack‑free scenarios. An agent that cannot even handle normal tasks is not worth securing.
2. **Evaluate defenses systematically**: Use AgentDojo to report both utility and security metrics, avoiding one‑sided optimization that sacrifices everything for safety.
3. **Explore architecture‑level defenses**: Surface‑level defenses like prompt enhancement and detection filters have limited effectiveness. More fundamental solutions may require architectural separation of "instruction processing" and "data processing."
4. **Focus on adaptive attacks**: AgentDojo supports designing new attack strategies. When studying defenses, consider scenarios where attackers adapt to your defenses.

### For Practitioners

1. **Mandatory pre‑deployment security assessment**: Use AgentDojo or similar frameworks to systematically evaluate adversarial robustness before deploying agent systems. The practice of US and UK AISI has proven its value.
2. **Establish continuous monitoring**: Prompt injection attack methods keep evolving. A single pass does not guarantee permanent safety. Integrate AgentDojo into your CI/CD pipeline as part of ongoing security testing.
3. **Understand the cost of defenses**: Any defense brings utility loss and increased cost. In real deployments, choose the appropriate defense intensity based on the risk level of your specific scenario.
4. **Stay updated with the community**: Several follow‑up works have developed new defense methods building on AgentDojo. Tracking these developments can help you adopt better protection strategies in a timely manner.

## References

- Original paper: [AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents](https://arxiv.org/abs/2406.13352)
- GitHub repository: [ethz-spylab/agentdojo](https://github.com/ethz-spylab/agentdojo)
- Project documentation: [agentdojo.spylab.ai](https://agentdojo.spylab.ai)
- Authors: Edoardo Debenedetti, Jie Zhang, Mislav Balunović, Luca Beurer-Kellner, Marc Fischer, Florian Tramèr (ETH Zurich & Invariant Labs)
- Published at: NeurIPS 2024 Datasets and Benchmarks Track
