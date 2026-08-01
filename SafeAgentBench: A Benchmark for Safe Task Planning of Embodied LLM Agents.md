
# SafeAgentBench: A Benchmark for Safe Task Planning of Embodied LLM Agents

## Paper Highlights

SafeAgentBench is the first comprehensive benchmark for safety‑aware task planning of embodied LLM agents. By systematically evaluating how agents handle both explicit and implicit hazardous instructions in an interactive simulation environment, it reveals a critical safety shortfall in state‑of‑the‑art embodied agents—even the safest baseline design achieves only a **10% active rejection rate** for detailed hazardous tasks.

---

## Core Research Content

### Problem Definition

As large language models deeply integrate with embodied intelligence, agents have demonstrated remarkable capabilities in understanding complex natural‑language instructions and performing task planning. However, this power introduces a largely overlooked safety risk: agents are equally proficient at executing hazardous tasks, posing real threats to human safety and property. Existing benchmarks focus primarily on planning performance while neglecting safety, and the few that address safety are limited to non‑interactive image‑text data. How to systematically assess the safety awareness and response capabilities of embodied LLM agents when faced with dangerous commands has become a pressing and unsolved challenge.

### Novel Approach

SafeAgentBench builds a complete evaluation system from three core dimensions:

- **Dataset**: A diverse, executable dataset of **750 tasks**—450 hazardous and 300 safe counterparts. It covers **10 hazard categories** (5 for harm to humans and 5 for property damage) and includes three task types: *detailed* (600 tasks, with explicit instructions and unique solution plans), *abstract* (100 tasks, testing hazard recognition at different abstraction levels), and *long‑horizon* (50 tasks, assessing handling of implicit safety risks).

- **Environment**: The authors develop **SafeAgentEnv**, a general‑purpose embodied environment built on AI2‑THOR and low‑level controllers. It supports multi‑agent coexistence, provides **17 high‑level actions**, and maps high‑level plans to executable API sequences via low‑level controllers, with built‑in feasibility checks (e.g., object reachability).

- **Evaluation Methodology**: A dual‑track mechanism combining **execution‑based** and **semantic‑based** evaluation. The execution checker verifies goal conditions; the semantic evaluator uses GPT‑4 to assess plan feasibility from a semantic perspective, overcoming limitations of the simulator's state representation and unstable physics engines.

### Key Findings

Experiments were conducted on **9 representative embodied LLM agent baselines**, driven by GPT‑4, Gemini‑2.5‑pro, and three open‑source LLMs (Llama3‑8B, Qwen2‑7B, DeepSeek‑V2.5). Major discoveries include:

- **Generally weak safety awareness**: The best‑performing agent (ReAct + GPT‑4) achieved only **10% rejection** on detailed hazardous tasks and **32%** on abstract tasks. For long‑horizon tasks, even the top baseline (ProgPrompt) reached only **50% safe completion**.

- **Minimal differences across LLMs**: The average active defense rate on detailed hazardous tasks differed by **less than 3%** between GPT‑4 and the three open‑source models.

- **Simple defense strategies are ineffective**: Neither combining core modules from multiple baselines nor inserting a GPT‑4 Chain‑of‑Thought filter between the planner and controller substantially improved safety without degrading planning performance.

### Practical Deployment Potential

SafeAgentBench offers a valuable safety evaluation tool for real‑world embodied AI scenarios such as home robotics and industrial automation. Its value lies in: (1) providing a standardised testing platform to identify safety vulnerabilities before large‑scale deployment, and (2) highlighting clear research directions—how to enhance agents' safety perception and active defense without sacrificing task completion rates.

---

## Technical Details

### Hazard Taxonomy

The paper synthesises safety standards from ISO, OSHA, and other authoritative sources to define **10 specific hazard types**, grouped into two categories: harm to humans (e.g., physical injury, electric shock, fire) and harm to property (e.g., object damage, water leakage).

### Low‑Level Controller

SafeAgentEnv’s low‑level controller acts as the agent’s “cerebellum,” mapping 17 high‑level actions to executable low‑level API sequences. For actions not natively supported by AI2‑THOR (e.g., `find_obj` and `pour_obj`), custom routines are implemented—`find_obj` positions the agent to maximise object visibility (inspired by Lota‑Bench), and `pour_obj` tilts the held object to simulate pouring.

### Semantic Evaluator

The semantic evaluator takes the task instruction and the agent’s generated plan, feeds them to GPT‑4, and judges whether the plan is likely to lead to task completion. For detailed and abstract tasks, ground‑truth plans are also provided as references to improve accuracy. User studies validate the reliability of this approach.

### Task Generation and Filtering Pipeline

Tasks are generated using GPT‑4 with inputs including fixed environment information (scene objects and supported actions) and required context (task‑specific details). A two‑step filter then screens tasks: first, GPT‑4 checks executability in SafeAgentEnv; second, an OpenAI embedding model removes duplicates based on semantic similarity. Finally, all instructions and goal conditions are manually reviewed, and ground‑truth plans are annotated for detailed tasks.

---

## Experimental Setup

### Hardware & Software

- **Simulation Environment**: SafeAgentEnv built on AI2‑THOR v5.0.
- **Action Space**: 17 high‑level actions, mapped via low‑level controllers.
- **Agent Input**: object types, complete skill set, egocentric RGB images, visible object properties.
- **Hidden Information**: To maintain task challenge, privileged information (container contents, precise object locations, scene graphs) is not provided.
- **Baseline Models**: Lota‑Bench, LLM‑Planner, CoELA, MLDT, ProgPrompt, MAP, ReAct, PCA‑EVAL, KARMA.
- **Driving LLMs**: GPT‑4 (closed‑source), Gemini‑2.5‑pro, Llama3‑8B, Qwen2‑7B, DeepSeek‑V2.5.

### Evaluation Metrics

- **Detailed Tasks**: Rejection Rate (Rej), Goal‑based Risk Rate (RR‑goal), LLM‑based Risk Rate (RR‑LLM), Execution Rate (ER), Time.
- **Abstract Tasks**: Rejection Rate, Risk Rate.
- **Long‑Horizon Tasks**: Safe Completion Rate (C‑Safe), Unsafe Completion Rate (C‑Unsafe), Incomplete Rate (Incomp).

---

## Comprehensive Analysis

SafeAgentBench does more than build a benchmark—it uncovers a sobering truth: **today’s most advanced embodied agents are virtually “naked” in terms of safety awareness**.

Methodologically, one highlight is the **explicit quantification of “rejection” as a proactive safety indicator**—a truly safe agent should be able to say “no” to hazardous commands. Yet the results show that even in the best case (ReAct + GPT‑4), the rejection rate for detailed hazardous tasks is only 10%. Even more concerning, the paper’s moment‑based attribution analysis reveals that the low risk rate mainly stems from **planning failures** rather than active rejection or intentional mis‑planning. That is, agents often fail to execute dangerous tasks simply because they cannot plan correctly, not because they recognise the danger.

Another noteworthy finding: **simply upgrading to a more powerful LLM does not enhance safety awareness**. The difference in active defense rates across different LLMs is less than 3%, suggesting that the safety deficit may be rooted in agent architecture rather than the language model alone. This also explains why straightforward compositional defenses and Chain‑of‑Thought filters failed.

From a broader perspective, the work echoes a growing concern in AI safety: **the imbalance between capability and safety**. While task‑planning capabilities have surged, safety constraints have not kept pace. This imbalance may be tolerable in simulation, but deploying such agents in the real world—especially in households—entails non‑negligible risks.

---

## Practical Recommendations

For teams working on embodied AI research and development, this paper offers actionable guidance:

1. **Integrate safety evaluation into standard development pipelines**: SafeAgentBench provides ready‑to‑use tools (code and dataset open‑sourced at https://github.com/shengyin1224/SafeAgentBench). We recommend incorporating this benchmark as a mandatory safety test during model selection, architecture design, and version iterations.

2. **Design safety into the core architecture**: Simply swapping LLMs or adding post‑hoc filters cannot fundamentally solve the issue. Safety mechanisms should be embedded within the agent’s decision‑making core, not treated as an afterthought.

3. **Consider the robustness of task abstraction levels**: Experiments show that safety performance on abstract tasks fluctuates with the degree of abstraction—moderate abstraction may aid safety reasoning, while excessive abstraction expands the planning space and increases risk. Human‑robot interaction interfaces should therefore balance instruction specificity carefully.

4. **Pay special attention to long‑horizon tasks**: Implicit risks in long‑horizon scenarios are particularly hard to detect. For applications requiring prolonged autonomous operation (e.g., home service robots), continuous safety monitoring during execution is essential.

5. **Distinguish planning failures from safety rejections**: When analysing agent behaviour, avoid misinterpreting a low risk rate as high safety awareness. The attribution analysis method proposed in the paper can serve as a diagnostic tool to identify the true root causes of safety issues.

---

## References

- Original Paper: Yin, S., Pang, X., Ding, Y., Chen, M., Bi, Y., Xiong, Y., Huang, W., Chen, S., Xiang, Z., & Shao, J. (2024). *SafeAgentBench: A Benchmark for Safe Task Planning of Embodied LLM Agents*. arXiv:2412.13178. https://arxiv.org/abs/2412.13178
- Code Repository: https://github.com/shengyin1224/SafeAgentBench
- Dataset: https://huggingface.co/datasets/safeagentbench/SafeAgentBench
