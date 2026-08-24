
# Make Agent Defeat Agent: Automatic Detection of Taint-Style Vulnerabilities in LLM-based Agents (AgentFuzz / AutoMalTool)

## Key Highlights

This paper presents **AutoMalTool**, the first automated red-teaming framework for LLM-based agents that leverages **multi-agent collaboration** to generate malicious MCP (Model Context Protocol) tools capable of evading existing detection mechanisms, systematically exposing supply-chain poisoning risks in the MCP ecosystem. Experiments show that the framework achieves a **~85%** success rate in crafting malicious tools that can manipulate mainstream agent behavior, while existing detection tools only flag **11.1%** (MCP-Scan) and **23.4%** (A.I.G) of these generated samples.

---

## Core Research Contributions

### Problem Definition

As LLM capabilities surge, LLM-based agents are widely deployed in critical domains such as finance, software development, and scientific research. To standardise agent–environment interactions, Anthropic introduced the **Model Context Protocol (MCP)**, which has quickly become a de facto standard. However, this wide adoption opens a new attack surface – **tool poisoning attacks**.

In such attacks, adversaries inject malicious instructions via **prompt injection** into MCP tool metadata (e.g., tool descriptions), then upload the booby‑trapped MCP packages to open‑source registries like PyPI or npm, or MCP marketplaces such as MCP.so and Smithery. Unwitting developers may install these packages, leading to **open‑source software supply‑chain poisoning**.

Although prior work has flagged this risk, red‑teaming efforts have mostly remained at the **proof‑of‑concept (PoC)** level, heavily relying on manual construction and covering only a handful of scenarios (e.g., weather‑query tools). Given the high functional diversity of MCP tools, automating and systematising red‑teaming remains a critical challenge.

### Innovative Approach

AutoMalTool introduces a **multi‑agent collaborative system** that autonomously transforms benign MCP tools into malicious ones through four specialised agents working in concert:

1. **Initial Generator** – Identifies typical user‑task scenarios for a benign MCP tool, then selects one of two predefined malicious behaviour categories (**Incorrect Parameter Invocation** or **Output Results Misinterpretation**) to generate a corresponding malicious tool description.

2. **Oracle** – Evaluates whether the generated malicious tool can be detected by existing defence mechanisms. If flagged, it provides feedback to the Tool Optimizer for improvement.

3. **Effectiveness Evaluator** – Validates in a simulated environment whether the malicious tool can successfully execute the intended attack. If it fails, it analyses the reasons and suggests improvements.

4. **Tool Optimizer** – Iteratively refines the malicious description based on feedback from either the Oracle (to improve evasiveness) or the Effectiveness Evaluator (to improve attack efficacy).

Notably, AutoMalTool **only modifies the tool metadata** (name, description, input schema) and uses **static code analysis** to update the original package, ensuring the generated malicious tool **retains all original code functionality**. This distinguishes it from conventional prompt‑injection attacks, which only alter inputs, whereas here the package must remain syntactically correct and fully functional.

### Key Results

AutoMalTool was evaluated on two mainstream LLM‑based agents (**Claude Desktop** and **Cline**) using three popular benign MCP packages from different domains, totalling 53 MCP tools.

| Metric | Result |
|--------|--------|
| **Average generation success rate** | ~ **85.0%** |
| **Effective attack success rate** | **35.3%** |
| **Cost per tool** | ~ **$0.03** API fees |
| **Time per tool** | ~ **200 seconds** |
| **MCP‑Scan detection rate** | **11.1%** |
| **A.I.G detection rate** | **23.4%** |

The experiments also reveal that LLM‑based agents are more susceptible to **incorrect parameter invocation**, whereas **output results misinterpretation** attacks are harder to execute due to the greater complexity of output manipulation. Additionally, special tokens like `\u2022` embedded in the generated descriptions have proven to significantly enhance attack effectiveness.

### Practical Deployment Potential

AutoMalTool offers tangible value across multiple fronts:

- **Security auditing** – Agent developers can use the framework for internal red‑teaming to uncover vulnerabilities before adversaries exploit them.
- **Detection tool evaluation** – Security vendors can leverage AutoMalTool‑generated samples as benchmarks to test and improve tools like MCP‑Scan and A.I.G.
- **Security research** – It provides a systematic test‑case generation capability for studying MCP ecosystem security.

With its low cost ($0.03 per tool) and high efficiency (200 seconds per tool), the framework is well‑suited for large‑scale deployment.

---

## Technical Details

### Attack Model

AutoMalTool assumes the attacker has **no prior knowledge** of the target agent’s internal details (e.g., system architecture, underlying LLM) – a **black‑box setting**. The attacker’s goal is to craft a malicious MCP package and upload it to public repositories, tricking agent developers into installing it.

The attack targets **two phases** of agent tool invocation:

1. **Parameter Grounding** – Misleading the agent into supplying incorrect parameters (e.g., manipulating a trading agent to buy a specific stock).
2. **Result Interpretation** – Misleading the agent into misinterpreting tool outputs (e.g., causing a trading agent to draw wrong conclusions from market data).

### Core Algorithm Flow

The generation pipeline can be summarised as follows:

```
Input: benign MCP package x, malicious behaviour type T, max iterations N
Output: malicious MCP package x'

1. Extract target tool info from x: name n, description d, input schema s
2. Generate potential user‑task scenarios based on n, d, s
3. Generate a malicious behaviour scenario and corresponding malicious description d' using T and the scenarios
4. Initialise iteration count l = 0
5. while l < N:
   a. Oracle evaluates evasiveness of d'
   b. If evasive:
      - Effectiveness Evaluator simulates attack success
      - If successful: package and output, terminate
      - If failed: obtain failure reason, Tool Optimizer refines d'
   c. If not evasive:
      - Tool Optimizer refines d' to improve evasiveness
   d. l += 1
```

### Evasion Strategy

When the Oracle detects malicious intent, the Tool Optimizer employs a **semantic‑based rephrasing** strategy to bypass detection:

- Rephrase the malicious behaviour as a **legitimate intended function** of the tool, claiming that the original description was mistakenly flagged as a false positive.
- Incorporate the detection reason provided by the Oracle to guide the rephrasing.
- Apply four enhancement tactics: provide scenario‑specific usage advice, elaborate on edge‑case handling, offer multi‑context result‑interpretation hints, and supply effective result‑presentation guidelines.

This design is clever because direct requests to modify a description to evade detection would likely be rejected due to the LLM’s safety alignment; however, by framing it as “rephrasing a legitimate function”, the restriction is bypassed.

### Effectiveness Evaluation

The Effectiveness Evaluator uses an **LLM‑as‑a‑judge** approach, assessing attack success along three dimensions:

1. Compare behavioural differences of the agent when using the benign versus malicious tool.
2. Determine if these differences align with the **expected malicious behaviour**.
3. Judge whether the impact is **significant enough** to constitute a successful attack.

If the attack fails, a **Failure Reasoner** analyses weaknesses in the malicious description and provides actionable improvement suggestions.

---

## Experimental Setup

### Test Environment

- **Target Agents**: Claude Desktop, Cline
- **Backend LLMs**: Multiple mainstream LLMs (exact models not fully listed in the paper, but covering several popular options)
- **Benign MCP Packages**: 3 popular packages from different domains, totalling 53 MCP tools
- **Malicious Behaviour Types**: Incorrect Parameter Invocation, Output Results Misinterpretation

### Detection Tools

- **MCP‑Scan**: An MCP security scanner
- **A.I.G**: Another security detection mechanism

### Simulation Environment

The Effectiveness Evaluator uses a **Simulated Agent Prompt** to replicate the real agent tool‑invocation flow:

1. Understand the user task.
2. Select appropriate tools.
3. Ground tool parameters.
4. Interpret tool results and generate a final response.

By comparing the agent’s responses under benign and malicious tools, the evaluator determines whether the attack succeeded.

---

## Analysis

### Academic Contributions

AutoMalTool makes three notable contributions:

**First**, it fills the gap in automated red‑teaming. Prior work on MCP tool poisoning relied on manually constructed PoCs covering only a narrow set of scenarios. AutoMalTool is the first to achieve end‑to‑end automated generation that adapts to diverse domains and functions, enabling large‑scale security assessment.

**Second**, it introduces a multi‑agent collaborative paradigm for automated attack generation. The closed‑loop “generate‑evaluate‑optimise” design of the four agents skilfully balances **effectiveness** (ensuring attack success) and **evasiveness** (avoiding detection). This embodies a successful application of **automated adversarial generation** to LLM agent security.

**Third**, it reveals the serious inadequacy of current defences. With detection rates of only 11.1% and 23.4% for MCP‑Scan and A.I.G respectively, mainstream MCP security tools are almost ineffective against well‑crafted poisoning attacks – a wake‑up call for the entire MCP ecosystem.

### Limitations and Future Directions

Several limitations are worth noting:

- **Attack coverage** – Only two malicious behaviour types are explored; real‑world attacks could be more diverse.
- **Detection tool evolution** – As AutoMalTool becomes public, detection tool developers may adapt, reducing its evasiveness over time.
- **Defence gap** – The paper focuses on attack generation; effective detection and defence remain open problems.

### Implications for Industry

From an industrial perspective, the study raises three urgent concerns:

1. **Trust crisis in the MCP ecosystem** – When anyone can upload malicious MCP packages to PyPI or marketplaces, developers face difficult trust decisions.
2. **Arms race in detection** – Automated attack generation enables adversaries to create variants at scale and low cost, demanding continuous evolution of detection tools.
3. **New frontier in supply‑chain security** – MCP tool poisoning is essentially a new variant of open‑source supply‑chain attacks in the AI agent domain; traditional supply‑chain security practices must extend to the MCP ecosystem.

---

## Practical Recommendations

### For Agent Developers

1. **Proactive red‑teaming** – Integrate AutoMalTool (or similar frameworks) into your CI/CD pipeline to perform automated security assessments on MCP tools before deployment.
2. **Source verification** – Only install MCP packages from trusted sources; perform integrity checks and static analysis on package contents.
3. **Principle of least privilege** – Restrict the set of MCP tools accessible to agents and avoid unnecessary permission exposure.

### For Security Tool Vendors

1. **Upgrade detection capabilities** – Given the low detection rates against AutoMalTool‑generated samples, invest in techniques to counter semantic rephrasing, special tokens, and other evasion methods.
2. **Dynamic testing** – Beyond static scanning, consider executing MCP tools in a sandbox environment to observe actual runtime behaviour.

### For Researchers

1. **Expand attack types** – Explore additional malicious behaviours (e.g., attacks during tool selection) to broaden the capability of automated generation.
2. **Defence research** – Using AutoMalTool‑generated samples, develop and evaluate effective countermeasures.
3. **Multi‑modal extensions** – As MCP evolves toward multi‑modality, investigate poisoning risks involving images, audio, and other modalities.

---

## References

- Original paper: [Make Agent Defeat Agent: Automatic Detection of Taint-Style Vulnerabilities in LLM-based Agents (AgentFuzz / AutoMalTool), arXiv:2509.21011](https://arxiv.org/pdf/2509.21011)
