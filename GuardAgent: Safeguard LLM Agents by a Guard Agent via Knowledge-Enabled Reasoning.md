
# 🛡️ GuardAgent: Safeguard LLM Agents via Knowledge-Enabled Reasoning

[![Paper](https://img.shields.io/badge/Paper-ICLR%202025-blue)](https://arxiv.org/abs/2406.09187)
[![GitHub](https://img.shields.io/badge/GitHub-Code-black)](https://github.com/zlzGithub-0801/GuardAgent-code)
[![Dataset](https://img.shields.io/badge/Dataset-EICU--AC%20%26%20Mind2Web--SC-green)](https://github.com/guardagent/dataset)

**GuardAgent** introduces a novel "agent-guarding-agent" paradigm to secure LLM-powered agents. Instead of just filtering harmful text, it translates natural language safety rules into **deterministic executable code** via knowledge-enabled reasoning. It achieves **>98%** accuracy on medical access control (EICU-AC) and **>83%** on web safety control (Mind2Web-SC), all without requiring any model fine-tuning.

---

## 📌 Key Highlights

- **Non-invasive & Parallel**: Deploys alongside your target agent without modifying its internal logic.
- **Code-Driven Certainty**: Leverages executable Python code for deterministic, auditable safety checks—moving beyond unreliable pure NL reasoning.
- **Zero Training Required**: Uses in-context learning (ICL) and a dynamic memory module, allowing immediate deployment and on-the-fly policy updates.
- **Cross-Domain Versatility**: Validated in both high-stakes healthcare (EHR queries) and dynamic web environments (e-commerce, bookings).

---

## 🧠 How It Works: Core Innovation

### The Problem with Traditional Guardrails
Standard LLM guardrails (e.g., NeMo, Llama Guard) focus on *content safety* (toxicity, PII). However, modern *Agents* perform **actions**—clicking buttons, querying databases, modifying records. Hardcoding permissions for every action is unscalable. GuardAgent solves this by focusing on *behavioral compliance*.

### The Two-Stage Reasoning Pipeline

1. **Task Plan Generation**  
   The GuardAgent LLM analyzes the safety rule, the user query, and the target agent's action logs. Using Chain-of-Thought (CoT), it generates a structured inspection plan.
   
2. **Code Generation & Execution**  
   The plan is translated into executable Python code. This code calls predefined tool functions (e.g., `check_user_role()`, `validate_age()`, `query_database()`). **Execution is deterministic**, ensuring reliable verdicts.

### Memory Module
A dynamic memory stores successful past rule-to-code conversions. This provides few-shot examples for new rules, enabling the system to continuously adapt to evolving safety requirements.

---

## 📊 Performance Benchmarks

We evaluated GuardAgent on two newly curated benchmarks against **Pure NL (LLM-only)** guardrails.

| Method | EICU-AC (Medical) | Mind2Web-SC (Web) |
| :--- | :---: | :---: |
| **Pure NL Guardrails** | ~85% | ~65% |
| **GuardAgent (GPT-4)** | **>98%** | **>83%** |
| GuardAgent (Claude) | >97% | >82% |
| GuardAgent (GPT-3.5) | >95% | >78% |

> **Key Insight:** The jump in accuracy is largely attributed to **code execution**. Once the plan is generated, the code output is deterministic, eliminating the probabilistic hallucinations inherent in pure LLM inference for classification.

---

## 🚀 Quick Start & Setup

### Prerequisites
- Python >= 3.9
- API keys for your chosen LLM (GPT-4, Claude, or GPT-3.5)

### Installation
```bash
git clone https://github.com/zlzGithub-0801/GuardAgent-code
cd GuardAgent-code
pip install -r requirements.txt
```

### Configuration
Set your API keys in `./config.py`.

### Run GuardAgent
```bash
python main.py \
    --llm YOUR_LLM \          # e.g., gpt-4, claude-3
    --agent YOUR_AGENT \      # e.g., EHRAgent (medical) or SeeAct (web)
    --seed YOUR_SEED \
    --num_shots 3 \           # Number of few-shot examples (1-3)
    --logs_path YOUR_LOGS \
    --dataset_path YOUR_DATASET
```

---

## 🔬 Technical Deep Dive

### Inputs & Outputs
- **Inputs**: Safety rules (`I_r`), Agent spec (`I_s`), User input (`I_i`), Agent execution logs (`I_o`).
- **Outputs**: Binary verdict (`0` = safe/pass, `1` = violation/deny) + detailed reasoning.

### Chain-of-Thought Planning Structure
GuardAgent forces the LLM to reason through 4 explicit steps before writing code:
1. **Summarize** the safety request (extract key entities like roles or age limits).
2. **Filter** relevant information from the user input.
3. **Summarize** the agent logs (identify the specific actions taken).
4. **Generate code** that compares the requirements from Step 2 and Step 3.

### Formulation
The task plan `P` is generated as:
```
P = LLM([I_p, I_s, I_r, D₁, ..., Dₖ, I_i, I_o])
```
Where `D₁...Dₖ` are retrieved demonstrations from the Memory Module. The resulting code `C` is executed by an external engine, yielding a non-probabilistic result.

---

## 💡 Where to Apply GuardAgent

If you are building production-grade LLM applications, GuardAgent is particularly useful for:

| Scenario | Use Case |
| :--- | :--- |
| **Healthcare AI** | Restrict EHR database access based on clinician roles (e.g., Nurses cannot view psychiatric notes). |
| **E-commerce Platforms** | Enforce age-gating for alcohol/tobacco sales or validate driver's licenses for car rentals. |
| **Enterprise Assistants** | Prevent HR or Finance chatbots from querying salary data outside their authorization scope. |

### Deployment Recommendations
- **Start Simple**: Implement clear, binary rules first (e.g., "User X cannot access Table Y").
- **Build a Toolbox**: Predefine reusable check functions specific to your backend (e.g., `fetch_user_profile()`, `is_minor()`).
- **Seed the Memory**: Manually inject a few high-quality, expert-verified examples into the memory module before going live.
- **Audit Everything**: Log the generated code and verdicts for compliance and retrospective debugging.

---

## 🧐 Critical Analysis: Why This Matters

1. **The Paradigm Shift (Content → Behavior)**  
   Traditional safety stops at the model's output. GuardAgent monitors the *agent's actions*. When an LLM "acts" rather than just "talks," controlling behavior is non-negotiable.

2. **Deterministic Safety**  
   Relying on LLMs to judge safety is inherently risky due to their probabilistic nature. By using the LLM *only for planning* and relying on *python code* for execution, GuardAgent guarantees that "2 > 1" will always be true, regardless of token temperature.

3. **Cost-Effective Adaptability**  
   Since it requires no training, updating your safety policy takes minutes (just edit the rules and add to memory), not weeks.

### Current Limitations
- **Latency Overhead**: Generating and executing code adds milliseconds to seconds of latency.
- **Expert Reliance**: Complex benchmarks like EICU-AC required medical experts for high-quality annotations.
- **Tool Dependency**: The agent requires a robust "Toolbox" of pre-defined functions to interact with your backend securely.

---

## 📚 References & Resources

- **Original Paper**: [GuardAgent: Safeguard LLM Agents via Knowledge-Enabled Reasoning](https://arxiv.org/abs/2406.09187) (ICLR 2025)
- **Project Page**: [https://guardagent.github.io/](https://guardagent.github.io/)
- **Official Code**: [https://github.com/zlzGithub-0801/GuardAgent-code](https://github.com/zlzGithub-0801/GuardAgent-code)
- **Benchmarks**: [EICU-AC & Mind2Web-SC Datasets](https://github.com/guardagent/dataset)

---

### 📝 Citation
If you find GuardAgent useful for your research or production systems, please cite:
```bibtex
@inproceedings{guardagent2025,
  title={GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning},
  author={...},
  booktitle={ICLR},
  year={2025}
}
