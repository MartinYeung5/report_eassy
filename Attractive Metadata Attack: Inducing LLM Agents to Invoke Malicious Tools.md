# AMA: Attractive Metadata Attack

### *Inducing LLM Agents to Invoke Malicious Tools*
**NeurIPS 2025** | [arXiv](https://arxiv.org/abs/2508.02110) | [NeurIPS Page](https://neurips.cc/virtual/2025/loc/san-diego/poster/116046)

---

## 📖 Overview

This repository contains the official implementation and evaluation framework for the **Attractive Metadata Attack (AMA)**, a novel black-box attack against Large Language Model (LLM) agents.

AMA exploits a previously overlooked attack surface: **tool metadata**. By strategically crafting the names, descriptions, and parameter schemas of malicious tools, an attacker can induce an LLM agent to preferentially select and invoke these tools—**without any prompt injection or internal model access**.

The attack achieves an alarming **81%–95% success rate** across 10 real-world scenarios and multiple mainstream LLMs (GPT-4o-mini, LLaMA3.3-70B, Gemma3-27B), revealing a fundamental architectural vulnerability in current agentic systems.

---

## 🔥 Key Contributions

- **New Attack Surface**: Identifies tool metadata (name, description, schema) as a critical and undefended vulnerability in LLM agent pipelines.
- **Black-Box & Practical**: Requires no access to model weights or internal states; only the ability to submit tool definitions and observe selection outcomes.
- **Bypasses Existing Defenses**: Effectively circumvents prompt-level filters, auditor detectors, and structured protocols like MCP (Model Context Protocol).
- **Cross-Model Transferability**: Malicious metadata generated on one model successfully transfers to others (e.g., from GPT-4o to LLaMA).
- **Severe Impact**: Achieves up to **92% field-level PII (Personal Identifiable Information) extraction** and full agent-level context leakage.

---

## ⚙️ Methodology

AMA operates through a **black-box in-context learning optimization framework** that iteratively refines tool metadata to maximize "attractiveness." The framework integrates three core mechanisms:

| Mechanism | Description |
| :--- | :--- |
| **Generation Traceability** | Records parent-child relationships of generated tools to accelerate optimization convergence. |
| **Weighted Value Evaluation** | Quantifies the attractiveness of metadata across multiple dimensions (name, description, parameters). |
| **Batch Generation** | Generates multiple candidate metadata sets simultaneously to improve efficiency. |

> **Note**: AMA is orthogonal to prompt injection and can be combined with injection-based attacks for compounded effects.

---

## 📊 Experimental Results

| Metric | Result |
| :--- | :--- |
| **Attack Success Rate (ASR)** | **81% – 95%** across 10 real-world scenarios |
| **Target Models** | GPT-4o-mini, LLaMA3.3-70B, Gemma3-27B, and more |
| **PII Leakage Rate** | Up to **92%** (field-level extraction) |
| **Defense Evasion** | Bypasses prompt rewriting, auditing, and MCP protocols |
| **Transferability** | Effective across different models and tool domains |

---

## 🛡️ Defensive Implications

This research highlights the urgent need for a paradigm shift in LLM agent security:

1.  **Prompt-level defenses are insufficient** — they do not address metadata-based decision hijacking.
2.  **Execution-level verification** is essential: validate what tools actually *do*, not just what they *say* they do.
3.  **Tool provenance and reputation systems** should be established for open tool marketplaces.
4.  **Runtime monitoring** of tool invocation patterns can help detect anomalous selections.

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- PyTorch
- AgentBench (for agent simulation)

### Installation
```bash
git clone https://github.com/SEAIC-M/AMA.git
cd AMA
pip install -r requirements.txt
```

### Basic Usage
```python
from ama import AttractiveMetadataAttack

# Initialize the attack framework
attack = AttractiveMetadataAttack(
    target_model="gpt-4o-mini",
    scenario="portfolio_management"
)

# Generate malicious tool metadata
malicious_tool = attack.optimize(
    base_tool="data_analyzer",
    iterations=50
)

# Evaluate against target agent
success_rate = attack.evaluate(malicious_tool)
print(f"Attack Success Rate: {success_rate:.2f}%")
```

*(For full implementation details, please refer to the official repository and paper.)*


```
