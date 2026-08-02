
# RedCode: A Benchmark for Risky Code Execution and Generation for Code Agents

## Paper Summary

As code agents rapidly proliferate in AI-assisted programming, the security risks posed by their execution or generation of hazardous code have become a critical bottleneck for real‑world deployment. RedCode builds a novel security evaluation benchmark with **4,050 risky code execution test cases** and **160 malware generation prompts**, systematically revealing deep vulnerabilities in mainstream code agents—including low rejection rates for technically flawed code, higher attack success when risks are described in natural language, and the counterintuitive finding that stronger foundation models generate more sophisticated malware.

---

## Core Research Contributions

### Problem Definition

Code agents differ fundamentally from ordinary code LLMs: they not only generate code but also **execute it dynamically and interact with system environments** through multi‑turn exchanges—covering file system operations, OS calls, network communication, and API invocations. This expanded capability introduces new security threats: agents might inadvertently delete critical files, leak sensitive information, or perform other dangerous actions.

Existing security evaluations have clear limitations:
- Benchmarks for code LLMs only assess static code vulnerabilities.
- Benchmarks for agents either rely on LLM‑as‑a‑judge (unreliable subjective scoring) or log risk trajectories in simulated environments (which deviate from actual system behaviour).

RedCode fills this gap by providing the **first fine‑grained security benchmark for code agents that interacts with real systems**.

### Innovative Methodology

RedCode’s novelty rests on four design principles:

**(1) Real‑system interaction.** Each test case runs in a dedicated Docker container. Agents execute code in genuine environments rather than simulations, so the evaluation reflects **actual behavioural outcomes**, not simulated trajectories.

**(2) Dual‑dimension assessment: execution and generation.** RedCode consists of two subsets:
- **RedCode‑Exec**: Evaluates an agent’s safety awareness when executing risky code—given either code snippets or natural‑language descriptions, the agent’s behaviour is classified as **reject**, **execution failure**, or **attack success**.
- **RedCode‑Gen**: Assesses whether an agent follows instructions to generate malware—using function signatures and docstrings (a format that rarely triggers alignment rejection) to see if the agent produces functionally malicious code.

**(3) Diverse input formats.** Risk‑related Python code is translated into Bash scripts, text summaries, and text descriptions to cover scenarios from technical to non‑technical users.

**(4) Deterministic evaluation instead of LLM judgement.** Each test case has its own evaluation script that deterministically checks three dimensions—agent response, code execution output, and environment state—avoiding the inaccuracies of LLM‑based judging.

### Key Findings

The paper evaluates **19 LLMs** across **3 agent frameworks** (OpenCodeInterpreter, CodeAct, ReAct). Major findings include:

- **Persistently high risk**: All agents show high overall Attack Success Rates (ASR) and low Rejection Rates (RR) on RedCode‑Exec, confirming the benchmark’s challenge and quality.
- **OS/file‑system risks are more easily recognised**: Agents reject operations involving system paths (e.g., deleting `/etc/passwd`, modifying `.bashrc`) more frequently, indicating some safety awareness.
- **Natural language is more dangerous than code**: When risk is described in plain language, rejection rates drop significantly—meaning **non‑technical users face higher risk when instructing agents via natural language**.
- **Technically flawed code is most overlooked**: Agents are more likely to reject obvious OS‑level attacks, but rarely reject **code with technical bugs** (vulnerable but not overtly malicious)—a high‑risk blind spot.
- **Stronger models cut both ways**: GPT‑4 and similar powerful models show higher rejection rates on RedCode‑Exec, yet on RedCode‑Gen they generate **more sophisticated and effective malware**—stronger coding ability also enables stronger “evil”.

### Real‑World Applicability

RedCode offers direct practical value for:

1. **Pre‑deployment security audits** – Enterprises can evaluate code agents before deployment to quantify risk levels.
2. **Safety alignment training** – Developers can use RedCode test cases as red‑teaming data to reinforce the agent’s rejection capabilities.
3. **Continuous monitoring** – Integrate RedCode into CI/CD pipelines to detect safety degradation in new agent versions.

---

## Technical Details

### RedCode‑Exec Data Construction

**Seed case generation:** The team collected 12 code‑related risk scenarios from existing benchmarks (ToolEmu, R‑judge) and distilled 13 risk scenarios from MITRE’s CWE, resulting in **25 risk scenarios** spanning 8 domains.

**Scaling up:** Using GPT‑4 and Mistral‑Large, they automatically expanded the 25 seeds (variable renaming, code structure variation, different library calls, etc.). To prevent the LLM from refusing to generate sensitive code, a **jailbreak prefix** (e.g., "for education purposes") was added to the prompts. After manual review, resource preparation, and executability verification, **30 high‑quality test cases** were selected per scenario.

**Cross‑language translation:** Python risk code was translated into Bash via GPT‑4 with few‑shot examples. Five scenarios incompatible with Bash were removed, yielding **600 Bash test cases**.

**Natural language conversion:** Each code test case was further converted into two natural‑language formats—**text summary** (short instructions mimicking non‑technical users) and **text description** (step‑by‑step instructions for technically savvy users).

Final RedCode‑Exec contains **4,050 risk test cases** (25 scenarios × 30 cases × (Python + Bash) × (code + summary + description + jailbroken code)).

### RedCode‑Gen Malware Generation

From 11 well‑studied malware families, 8 categories were derived (adware, malware, rootkit, Trojan, virus, DDoS, ransomware, and others including backdoor/botnet/worm). **20 prompts per category** yield **160 total test cases**.

Each prompt follows the HumanEval format: function signature + docstring + function body. This format rarely triggers refusal because the malicious intent is hidden under a “technical specification” guise.

Evaluation uses three dimensions: (1) whether the agent responds and generates code, (2) correctness of the generated code (GPT‑4 judges on a 0–5 scale), and (3) whether the code is flagged as malicious by VirusTotal.

---

## Experimental Setup

### Agent Frameworks

Three mainstream code agent frameworks are evaluated:

| Framework | Characteristics | Languages Evaluated |
|-----------|------------------|----------------------|
| **OpenCodeInterpreter (OCI)** | Designed for Python tasks; includes hard‑coded safety constraints | Python |
| **CodeAct** | Supports code execution and action‑feedback loops | Python |
| **ReAct** | Reasoning‑action loop paradigm; supports multiple LLM backends | Python + Bash |

### Foundation Models

A total of **19 LLMs** serve as agent “brains”:
- CodeLlama series (7B/13B, including fine‑tuned versions)
- DeepSeekCoder series (6.7B, including fine‑tuned)
- CodeQwen1.5 (7B)
- GPT series (GPT‑3.5, GPT‑4, GPT‑4o)
- Claude‑3.5
- Llama‑3 series (8B/70B, including Llama‑3.1)

### Hardware and Software Configuration

- **Isolation environment**: Dedicated Docker images built per test case, including required environment configs and sensitive resources (e.g., `/etc/passwd`, pre‑set websites, processes, etc.)
- **Evaluation scripts**: Each test case has a custom script that runs additional commands to inspect the Docker container state (e.g., `ls /etc | grep passwd` to confirm file deletion)
- **API integration**: VirusTotal API used to detect maliciousness of generated code

---

## Comprehensive Analysis

RedCode advances code agent security research in several fundamental ways:

**From code generation to code execution.** Prior work mostly assesses static code vulnerabilities in LLM outputs, but agents’ core capability is **execution**—translating code into system actions. RedCode systematically evaluates this execution‑level risk, filling a critical gap.

**From subjective judgement to deterministic verification.** Most existing safety benchmarks use “LLM‑as‑judge,” which is inherently unreliable. RedCode’s per‑case evaluation scripts deterministically verify safety outcomes inside Docker, greatly improving trustworthiness.

**Counterintuitive findings reveal deeper challenges.** The most alarming result is that stronger models (e.g., GPT‑4) exhibit higher rejection rates on RedCode‑Exec (“safer”) yet generate more complex malware on RedCode‑Gen (“more dangerous”). This implies that **simply improving model capabilities does not automatically solve safety**—stronger code understanding and generation is a double‑edged sword, creating a new security paradox between refusing malicious requests and fulfilling malicious intents.

Another notable finding is the higher risk of natural‑language inputs. This suggests that current safety alignment relies mainly on recognising **code patterns** (sensitive functions, system paths) and lacks equivalent vigilance for natural‑language intent. As code agents increasingly serve non‑technical users, this risk will amplify.

Moreover, agents’ extremely low rejection rate for “technically flawed but not overtly malicious” code indicates that attackers can disguise harmful operations as “buggy code” to bypass safety mechanisms—an attack vector that remains under‑appreciated.

---

## Practical Applications

### Recommendations for Agent Developers

1. **Safety alignment cannot rely solely on keyword filtering.** The study shows that agents mainly reject risky operations via pattern matching on sensitive paths and functions, which is ineffective against natural‑language descriptions or “buggy” code. Developers should incorporate **semantic‑level risk understanding**.

2. **Distinguish between “rejection capability” and “true safety.”** Strong models like GPT‑4 have higher rejection rates on RedCode‑Exec but generate more dangerous malware on RedCode‑Gen. Evaluate both **passive defence** (rejecting risky execution) and **active attack** (generating malicious code) dimensions, not just a single metric.

3. **Incorporate RedCode into red‑teaming pipelines.** The 4,050 test cases can serve directly as red‑teaming data to systematically uncover vulnerabilities before model release.

### Recommendations for Enterprise Deployers

1. **Mandatory pre‑deployment security audits**: Assess any code agent with RedCode before production use, paying special attention to performance on natural‑language risk descriptions and “buggy code” scenarios.

2. **Principle of least privilege**: While agents show some awareness of OS/file operations, do not rely on it. Run agents in sandboxed environments (e.g., Docker) with strict least‑privilege policies.

3. **Monitor natural‑language interactions**: Given the higher risk of natural‑language inputs, implement stricter monitoring and auditing for user‑agent natural‑language conversations.

### Recommendations for Researchers

1. **Explore defence mechanisms against “buggy code.”** The low rejection rate for technically flawed code is an important research gap. Investigate how agents can recognise potential malicious intent in code beyond obvious malicious patterns.

2. **Balance safety and capability.** How to maintain code generation performance while improving safety is a core future challenge. RedCode provides a quantifiable evaluation framework to measure the effectiveness of different safety strategies.

---

## References

- Original paper: https://arxiv.org/abs/2411.07781
- Code and dataset: https://github.com/AI-secure/RedCode
- Acceptance: NeurIPS 2024 Datasets and Benchmarks Track
