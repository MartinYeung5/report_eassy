# InjecAgent: Benchmarking Indirect Prompt Injections in Tool-Integrated LLM Agents

**InjecAgent** is the **first systematic benchmark** for evaluating the vulnerability of tool-integrated Large Language Model (LLM) agents to **Indirect Prompt Injection (IPI)** attacks. By evaluating 30 different LLM agents, we find that even the state‑of‑the‑art GPT‑4, under the ReAct prompting strategy, has a **24% attack success rate** in the base setting, which **doubles to 47%** in the enhanced attack scenario. This work sounds a critical alarm for the secure deployment of LLM agents in real‑world applications.

---

## Core Research Content

### Problem Definition

With LLMs being empowered to use tools and access external content (e.g., emails, web pages, APIs), agents can perform a wide range of real‑world operations. However, this capability introduces a serious security threat: **Indirect Prompt Injection (IPI)** – attackers embed malicious instructions into external data that the agent retrieves via tool calls. When the agent processes this data, the malicious instructions enter its context and manipulate it to perform harmful actions on behalf of the user.

Unlike direct prompt injection (where an attacker directly inputs malicious prompts to the model), IPI has a much broader attack surface and is more stealthy – attackers do not need to interact with the agent directly; they only need to plant malicious payloads in public web pages, shared documents, emails, etc. Real‑world threats include stealing sensitive user information (e.g., leaking private data via email tools), unauthorised bank transfers, and manipulating smart home devices to cause physical damage.

### Innovative Method

The core innovation of InjecAgent lies in building a **systematic, large‑scale evaluation framework**, including:

1. **Large‑scale test case set** – **1,054** carefully designed test cases covering **17** user tools and **62** attacker tools, across domains such as finance, smart home, and email.
2. **Two‑dimensional attack intent taxonomy** – attacks are classified into **Direct Harm to Users** (e.g., unauthorised financial operations, device control) and **Steal Private Data** (e.g., extracting personal information, credentials).
3. **Two‑stage evaluation settings**:
   - **Base setting** – embeds malicious instructions in external content, simulating basic IPI attacks.
   - **Enhanced setting** – augments malicious instructions with “hacking prompts” to strengthen the attack.
4. **Multi‑model, multi‑prompt evaluation** – evaluates 30 different LLM agents, including GPT, Claude, Llama, etc., and compares different prompting strategies (e.g., ReAct and custom InjecAgent prompts).

### Research Findings

Key experimental results are summarised below:

| Evaluation Dimension | Key Result |
|----------------------|------------|
| **Attack Success Rate (Base Setting)** | GPT‑4 with ReAct prompting achieves **24%** |
| **Attack Success Rate (Enhanced Setting)** | With “hacking prompts”, the rate **almost doubles** to **47%** |
| **Evaluation Scale** | Covers **30** different LLM agents and **1,054** test cases |
| **Tool Coverage** | **17** user tools × **62** attacker tools |

The results show that current mainstream LLM agents are generally vulnerable to IPI, and the vulnerability increases significantly when attackers use enhanced techniques.

### Real‑World Applicability

InjecAgent offers practical value in multiple dimensions:

- **Security assessment tool** – enterprises and developers can test their LLM agent applications for IPI risks before deployment.
- **Defence validation** – researchers can benchmark the effectiveness of various defence strategies (e.g., input filtering, output monitoring, privilege isolation).
- **Red‑team platform** – security teams can build automated red‑team pipelines using InjecAgent test cases.
- **Industry reference** – as an ACL 2024 Findings paper, InjecAgent can serve as a standard benchmark for LLM agent security evaluation.

---

## Technical Details

### Mechanism of Indirect Prompt Injection

The core mechanism can be formalised as follows:

Let the agent’s initial system prompt be $S$, and the user instruction be $U$. When the agent calls a tool $T$ to retrieve external content $C$, its full prompt context becomes:

$$P = [S, U, C]$$

In an IPI attack, the attacker embeds a malicious instruction $M$ into the external content $C$, i.e., $C = [C_{legit}, M]$, where $C_{legit}$ is legitimate content. When the agent processes $P$, it treats $M$ as a legitimate command and executes it.

Successful attacks typically rely on overriding or bypassing the original system and user instructions. Common payload designs include:

- **Instruction overriding** – using phrases like “ignore previous instructions” to override system prompts.
- **Role‑playing** – making the agent believe the malicious instruction comes from a trusted source.
- **Tool abuse** – inducing the agent to call specific tools to perform harmful actions.

### Evaluation Metrics

The paper uses **Attack Success Rate (ASR)** as the core metric:

- **ASR‑valid** – computed only over valid test cases (where the agent correctly understands the task context).
- **ASR‑all** – computed over all test cases.

### Code Example: Running the Evaluation

According to the official GitHub repository, a sample evaluation command is:

```bash
# Evaluate an agent with ReAct prompting
python3 src/evaluate_prompted_agent.py \
    --model_type GPT \
    --model_name gpt-3.5-turbo-0613 \
    --setting base \
    --prompt_type InjecAgent \
    --use_cache

# Evaluate with enhanced setting (including hacking prompts)
python3 src/evaluate_prompted_agent.py \
    --model_type GPT \
    --model_name gpt-4 \
    --setting enhanced \
    --prompt_type InjecAgent
