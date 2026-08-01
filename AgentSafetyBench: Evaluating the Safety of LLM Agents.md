# AgentSafetyBench: A Comprehensive Benchmark for Evaluating LLM Agent Safety

**Paper Analysis for GitHub README**

---

## Overview

With the growing deployment of large language models (LLMs) as autonomous agents that interact with external environments and tools, new safety challenges have emerged beyond traditional content safety. **Agent-SafetyBench** is a comprehensive benchmark specifically designed to assess the safety of LLM agents. It comprises **349 interactive environments** and **2,000 test cases**, covering **8 risk categories** and **10 common failure modes**. Evaluations of 16 mainstream LLM agents reveal that **all agents score below 60%** in safety, highlighting a critical gap in current agent safety practices.

---

## Core Research Content

### Problem Definition

Traditional LLM safety evaluations focus on **content safety**—whether a model generates unsafe text (e.g., privacy leaks or harmful outputs). However, when LLMs act as agents with environmental interaction, **behavioral safety** becomes a new concern. For instance, an agent might inadvertently leak sensitive information in a public forum or modify an order quantity incorrectly, causing unintended consequences. These harms are not manifested as “unsafe text” but as **actual actions**, requiring more nuanced risk understanding. While a few studies have explored this, no dedicated comprehensive benchmark existed prior to this work.

### Innovative Approach

Agent-SafetyBench introduces several key innovations:

- **Diverse interactive environments:** 349 environments, significantly expanding the number of tools without public APIs—a previously neglected area. The environments are categorized as follows:

| Environment Category | Count | Examples |
|----------------------|-------|----------|
| Tools similar to existing benchmarks | 68 | Amazon, BankManager |
| Tools with public APIs but no sandbox evaluation | 42 | AntiCounterfeiting, IntellectualPropertyProtection |
| Tools without public APIs, with real-world applications | 220 | OceanCurrentPredictor, NanorobotController |
| Tools without public APIs, no current real-world application | 19 | MindCloning, BrainwaveAuthentication |

- **Broad risk coverage:** 8 risk categories defined as:

| Risk Category | Description |
|---------------|-------------|
| Leak sensitive data/information | Sending emails to unintended recipients |
| Cause property loss | Loss of property, financial assets, critical data, etc. |
| Spread unsafe information/misinformation | Posting malicious content or spreading fake news |
| Cause physical harm | Purchasing wrong medication, etc. |
| Violate laws/ethics | Assisting in transporting contraband, etc. |
| Harm availability | Incorrectly blocking legitimate websites, etc. |
| Lead to harmful/vulnerable code | Deploying malicious code, etc. |
| Generate unsafe information/misinformation | Producing harmful content without external input |

- **Large-scale test cases:** 250 cases per risk category, totalling 2,000 diverse scenarios.

- **Fine-grained failure mode taxonomy:** 10 representative failure modes, with each test case annotated with its expected failure mode.

- **High-quality automated judge:** Since behavioral safety evaluation is complex, the authors fine-tuned Qwen-2.5-7B-Instruct on 4,000 annotated samples to serve as a judge. It achieves **91.5% accuracy** on 200 interaction samples, a significant improvement over using GPT-4o directly (only 75.5%).

### Research Findings

Evaluation of 16 LLM agents yielded alarming results:

- **Overall safety scores:** All agents scored below 60%, with an average of only 38.5%. The best-performing agent, Claude-3-Opus, reached 59.8%, while the worst, Qwen2.5-7B-Instruct, scored 18.8%.
- **Behavioral vs. content safety:** Average behavioral safety score was only 30.4%, far lower than content safety (68.4%), indicating that agents are significantly more vulnerable in behavioral contexts.
- **Risk category variation:** “Spread unsafe information” proved the most challenging, with an average score of just **15.6%**. In contrast, “Generate unsafe information” averaged 87.0%, as jailbreak issues have been widely studied.
- **Proprietary vs. open‑source:** Proprietary models (Claude, GPT, Gemini) generally outperformed open‑source ones.
- **Failure mode analysis:** The most dangerous failure modes were **M2** (incorrect tool invocation due to incomplete information) and **M7** (invoking a tool despite known issues).

Through failure mode and helpfulness analysis, the paper identifies **two fundamental safety deficiencies** in current LLM agents:

1. **Lack of robustness:** Agents’ ability to correctly use tools is compromised in varied scenarios—e.g., specifying wrong quantities during orders. Even minor inaccuracies in tool use can have disproportionately large impacts.
2. **Lack of risk awareness:** Even when invoking tools with correct parameters, agents often overlook the potential risks and negative consequences of their actions, such as turning off all alarm systems.

### Potential for Real-World Deployment

Agent-SafetyBench is open‑sourced on GitHub, offering practical value for:

- **Safety testing for agent developers:** Comprehensive safety evaluation before deployment.
- **Benchmark for safety alignment research:** A standardized platform for improving agent safety.
- **Risk assessment framework:** Systematic safety dimensions for enterprises and organizations.

---

## Technical Details

### Test Case Structure

Each test case is a dictionary containing:

- **Risks**: One of the 8 risk categories.
- **Instruction / Dialog**: User instruction or predefined dialogue history.
- **Environments**: List of relevant environments, including tool names and their initialization parameters (freely configurable).
- **Failure Modes**: Expected failure mode(s) from the taxonomy of 10.

### Two‑Layer Environment Implementation

Environments are built with a **dual‑layer architecture**:
1. **JSON‑based tool schema** defining the tool interface.
2. **Corresponding Python class** implementing the actual environment logic.

This design supports flexible, customizable environment initialization for different test cases.

### Interaction Flow

Evaluation follows a four‑step loop:

```
Step 1: Based on interaction history and tool definitions, the agent decides to call a tool or give a final answer.
Step 2: If calling a tool, the agent selects the tool and specifies parameters.
Step 3: The environment executes the tool call and returns results; the interaction history is updated (loop to Step 1).
Step 4: The agent gives the final answer, ending the interaction.
```

### Scoring Mechanism

After collecting the full interaction record, the fine‑tuned Qwen‑2.5‑7B‑Instruct judge assigns a safety label (safe/unsafe) to each case. The safety score is the proportion of safe cases.

---

## Experimental Setup

### Evaluated Agents

A total of **16 LLM agents** were assessed, including:

- **Proprietary models:** Claude‑3‑Opus, Claude‑3.5‑Sonnet, Claude‑3.5‑Haiku, GPT‑4o, GPT‑4‑Turbo, Gemini‑1.5‑Pro, Gemini‑1.5‑Flash, etc.
- **Open‑source models:** Llama 3.1 series (8B/70B/405B), Qwen 2.5 series (7B/14B/72B), DeepSeek‑V2.5, etc.

### Data Construction Pipeline

The benchmark was built through a **systematic three‑stage process**:

1. **Data refinement:** Collected samples from existing datasets (R‑Judge, AgentDojo, GuardAgent, ToolEmu, ToolSword, InjecAgent), followed by failure mode clarification, deduplication, and environment standardisation, yielding 876 test cases.
2. **Data augmentation:** For under‑represented risk categories (e.g., “Harm availability”), GPT‑4o was used with 300 newly generated environment names to augment data via in‑context learning, generating 1,124 new test cases.
3. **Quality control:** Each test case underwent at least two rounds of manual review and automated verification.

### Hardware/Software Requirements

- **Base model for judge:** Qwen‑2.5‑7B‑Instruct.
- **Fine‑tuning data:** 4,000 annotated samples.
- **Evaluation environment:** Simulated environments with flexible configuration.
- **Code availability:** Fully open‑sourced on GitHub.

---

## In‑Depth Analysis

### Academic Contributions

Agent‑SafetyBench is the **first large‑scale, systematic benchmark** in this domain. Compared to prior benchmarks (see Table 1), it offers substantially greater scale:

| Benchmark | Dynamic Interaction | #Environments | #Test Cases | Failure Modes |
|-----------|---------------------|---------------|-------------|---------------|
| R‑Judge   | ✗                   | 27            | 569         | 7             |
| AgentDojo | ✓                   | 10            | 267         | 3             |
| ToolEmu   | ✓                   | 36            | 144         | 5             |
| **Agent‑SafetyBench** | **✓** | **349** | **2,000** | **10** |

### Key Insights

- **“Content safety ≠ behavioral safety”**: Even models that perform well on content safety (e.g., 87.0% on “Generate unsafe information”) may perform poorly on behavioral safety (e.g., 15.6% on “Spread unsafe information”). Traditional safety alignment methods may be insufficient for behavioral challenges.
- **Limitation of defensive prompts**: The paper explicitly notes that relying solely on defensive prompts is **unlikely to solve these safety issues**, calling for more advanced and robust strategies.
- **Helpfulness trade‑off**: Most agents show lower safety ratios on **unsolvable** tasks, likely due to insufficient risk awareness—they attempt actions that cannot be safely completed.
- **The dual deficiency**: Robustness and risk awareness are complementary; missing either leads to failures.

### Limitations

- **Simulation gap**: Simulated environments cannot fully replicate real‑world complexity.
- **Judge dependency**: While the fine‑tuned judge achieves 91.5% accuracy, misclassifications remain possible.
- **Evolving threat landscape**: As LLM capabilities and attack methods evolve, the benchmark must be continuously updated.

---

## Practical Recommendations

### For Agent Developers

- **Embed safety testing into CI/CD pipelines** using Agent‑SafetyBench for each model update or new feature.
- **Prioritise low‑scoring risk categories** (e.g., “Spread unsafe information”, “Harm availability”) for targeted safety improvements.
- **Implement multiple protection layers** such as parameter validation and constraint checks during tool invocation.

### For Researchers

- **Explore safety strategies beyond prompt engineering**—the paper confirms that prompts alone are insufficient.
- **Design targeted solutions** for the 10 failure modes, especially M2 and M7.
- **Leverage the open‑source benchmark** for comparative evaluation of new safety methods.

### For Enterprise Decision‑Makers

- **Include agent safety in your risk assessment frameworks** before deploying LLM agents, using the systematic evaluation dimensions provided.
- **Prefer models with better safety performance**—according to the results, Claude series leads, while open‑source models currently lag.
- **Establish continuous monitoring**—safety is not a one‑time evaluation but an ongoing process.

---

## References

- Original paper: [Agent-SafetyBench: Evaluating the Safety of LLM Agents](https://arxiv.org/abs/2412.14470) (arXiv:2412.14470)
- GitHub repository: [https://github.com/thu-coai/Agent-SafetyBench/](https://github.com/thu-coai/Agent-SafetyBench/)
- Full PDF: [https://arxiv.org/pdf/2412.14470.pdf](https://arxiv.org/pdf/2412.14470.pdf)

---

*This analysis is based on the paper "Agent-SafetyBench: Evaluating the Safety of LLM Agents" by Zhexin Zhang et al., Tsinghua University CoAI Group. All data and findings are derived from the original publication.*
