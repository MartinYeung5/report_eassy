# 🛡️ BackdoorLLM: A Comprehensive Benchmark for Backdoor Attacks and Defenses on Large Language Models

> **NeurIPS 2025** | *Yige Li et al.* | Center for AI Safety (SafetyBench 1st Place)

[![NeurIPS](https://img.shields.io/badge/NeurIPS-2025-red)](https://neurips.cc/)
[![SafetyBench](https://img.shields.io/badge/SafetyBench-1st_Place-brightgreen)](https://www.safetybench.org/)
[![arXiv](https://img.shields.io/badge/arXiv-2408.12798-b31b1b)](https://arxiv.org/abs/2408.12798)
[![HuggingFace](https://img.shields.io/badge/HuggingFace-BackdoorLLM-yellow)](https://huggingface.co/BackdoorLLM)
[![GitHub](https://img.shields.io/badge/GitHub-Repo-blue)](https://github.com/bboylyg/BackdoorLLM)

## 📖 Overview

While backdoor attacks have been extensively studied in computer vision and text classification, their threat to generative Large Language Models (LLMs) remains critically under-explored. **BackdoorLLM** is the first comprehensive benchmark designed to systematically evaluate backdoor attacks and defenses on LLMs.

This work establishes a unified repository and standardized evaluation pipeline, covering **4 attack modalities**, **8 attack strategies**, **7 real-world scenarios**, **6 model architectures**, and over **200 experiments**. It reveals critical safety vulnerabilities and provides a solid foundation for reproducible research in LLM security.

---

## 🚀 Key Features

- **Unified Benchmark Framework**: Standardized training and evaluation pipelines ensuring fair comparisons across different methods.
- **Comprehensive Attack Coverage**: Integrates Data Poisoning, Weight Poisoning, Hidden State Attacks, and Chain-of-Thought (CoT) Hijacking.
- **Complete Defense Toolkit**: Built-in integration of 7 representative defense mechanisms.
- **Extensive Empirical Validation**: Over 200 controlled experiments across diverse models (Llama2-7b/13b, Mistral-7b, etc.).

---

## ⚙️ Technical Architecture

### Attack Modalities
| Category | Description |
| :--- | :--- |
| **Data Poisoning** | Injecting triggered malicious samples into training data. |
| **Weight Poisoning** | Directly modifying model parameters to embed backdoors. |
| **Hidden State Attacks** | Manipulating internal representations (activation steering). |
| **CoT Hijacking** | Interfering with the model's step-by-step reasoning process. |

### Defense Methods
The benchmark integrates 7 representative defense strategies, enabling a complete "Attack-Evaluation-Defense" closed-loop research system.

### Evaluation Setup
- **Models**: Llama2-7b, Llama2-13b, Mistral-7b, and more.
- **Scenarios**: Code generation, content moderation, question answering, etc.
- **Metrics**: Attack Success Rate (ASR), Clean Data Performance (CDP), and utility preservation.

---

## 🔑 Key Findings & Insights

1. **Universal Vulnerability**: Backdoor attacks are broadly feasible and effective across all tested LLM scales.
2. **Synergistic Risks**: Even ineffective backdoors can significantly enhance the success rate of jailbreak attacks.
3. **Scale Resilience**: Larger models exhibit stronger resistance to weight poisoning attacks.
4. **Hidden State Limitations**: Activation steering shows poor generalizability across different tasks.
5. **Reasoning Paradox**: Models with stronger reasoning capabilities are **more** vulnerable to CoT hijacking—revealing a fundamental tension between capability and safety.

---

## 💡 Deep Insights for the Community

> **Why this benchmark matters:**
BackdoorLLM serves as the "ImageNet" for LLM backdoor research. It moves the field from isolated, non-reproducible studies to a standardized, comparable, and extensible ecosystem.

- For **Researchers**: Provides a solid baseline. New attacks/defenses must be evaluated on this framework for fair comparison.
- For **Developers**: Offers a pre-deployment safety checklist. Helps identify vulnerabilities and choose appropriate defenses.
- For **Policymakers**: Supplies empirical data to support the formulation of LLM safety standards.

---

## 🗂️ Project Resources

- **📄 Paper**: [arXiv:2408.12798](https://arxiv.org/abs/2408.12798)
- **💻 Code**: [GitHub Repository](https://github.com/bboylyg/BackdoorLLM)
- **🤗 HF Community**: [HuggingFace/BackdoorLLM](https://huggingface.co/BackdoorLLM)
- **🌐 Project Page**: [Official Website](https://bboylyg.github.io/backdoorllm-website.github.io/)

### Related Extended Works
- **Backdoor4Good (B4G)**: Exploring beneficial uses of backdoor mechanisms.
- **BackdoorAgent**: Investigating backdoor threats in LLM-based agent systems.
- **AutoBackdoor**: Automated backdoor injection pipelines.

---
