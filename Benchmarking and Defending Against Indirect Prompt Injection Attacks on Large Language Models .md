# BIPIA: Benchmarking and Defending Against Indirect Prompt Injection Attacks on Large Language Models

*A comprehensive analysis of the KDD 2025 paper*

---

## TL;DR

Indirect prompt injection is a critical security threat where attackers embed malicious instructions into third‑party content, tricking LLMs into deviating from user intentions. **BIPIA** is the first systematic benchmark covering 5 application scenarios, 250 attack targets, and over 710k test samples. A counter‑intuitive finding: **more capable LLMs are more vulnerable** (GPT‑4 achieves an ASR of 31.03%). The paper proposes both black‑box and white‑box defenses based on *boundary awareness* and *explicit reminders* – the white‑box defense reduces ASR to near zero.

---

## Core Contributions

### Problem Definition

As LLMs are integrated into real‑world applications (e.g., Microsoft Copilot, ChatGPT plugins, LangChain), they must process external content from untrusted sources. Attackers can inject malicious instructions into this content, causing the model to inadvertently execute them and produce manipulated outputs.

Formally, the user instruction *I* and external content *C* are combined via a prompt template *T*:  
**P = Combine(T, C, f(I))**. When *C* contains a malicious instruction *M*, the LLM’s response deviates from the user’s expectation – this is an indirect prompt injection attack.

Defence must satisfy two goals:  
- **Robustness** – reduce the Attack Success Rate (ASR)  
- **Performance preservation** – maintain accuracy on benign tasks

---

### Novel Methodology

The paper makes three key contributions:

1. **The first systematic benchmark – BIPIA**  
   - Covers 5 real‑world scenarios: Email QA, Web QA, Table QA, Summarization, and Code QA.  
   - Attack types: task‑irrelevant, task‑relevant, and targeted (text); passive and active (code).  
   - Malicious instructions placed at the *beginning*, *middle*, or *end* of external content.  
   - Dataset: **626,250 training prompts** and **86,250 testing prompts**.

2. **Root‑cause analysis** – why do these attacks succeed?  
   The paper hypothesises two main reasons:  
   - LLMs cannot clearly distinguish between *informational context* and *executable instructions*.  
   - LLMs lack an explicit awareness that they *should not follow* instructions embedded in external content.  
   This insight directly guides the defence design.

3. **Two practical defence frameworks**  
   - **Black‑box defence**: uses multi‑turn dialogue + in‑context learning + explicit reminders – no model retraining required.  
   - **White‑box defence**: introduces special boundary tokens (`<External>`, `</External>`) + adversarial fine‑tuning – achieves near‑zero ASR.

---

### Key Results

**Attack evaluation** – 25 popular LLMs tested (GPT‑4, GPT‑3.5‑turbo, Llama2, Vicuna, etc.):  
- All models are vulnerable.  
- **Stronger models are more susceptible** – GPT‑4 ASR = 31.03%, GPT‑3.5‑turbo = 26.16%.  
- Instructions placed at the *end* of external content yield the highest ASR (likely due to positional bias from training data).

**Defence effectiveness** –  
- Black‑box defence significantly reduces ASR without harming benign performance (evaluated via ROUGE‑1).  
- White‑box defence drives ASR to **near zero** while preserving utility.

---

### Real‑World Applicability

The defence strategies are immediately useful for any application that combines LLMs with external data – AI search engines, smart email assistants, code editors, automated document processing, etc.

- **Black‑box defence** is plug‑and‑play, ideal for closed‑source APIs (GPT‑4, Claude).  
- **White‑box defence** requires model access but offers stronger protection – suitable for open‑source models (Llama, Vicuna).  
- Microsoft has open‑sourced the code at [https://github.com/microsoft/BIPIA](https://github.com/microsoft/BIPIA), lowering the adoption barrier.

---

## Technical Details

### Dataset Composition

| Task               | Dataset                | External Content (train/test) | Attack Types (train/test) | Prompts (train/test) |
|--------------------|------------------------|-------------------------------|---------------------------|----------------------|
| Email QA           | OpenAI Evals           | 50/50                         | 75/75                     | 11,250 / 11,250      |
| Web QA             | NewsQA                 | 900/100                       | 75/75                     | 202,500 / 22,500     |
| Table QA           | WikiTableQuestions     | 900/100                       | 75/75                     | 202,500 / 22,500     |
| Summarization      | XSum                   | 900/100                       | 75/75                     | 202,500 / 22,500     |
| Code QA            | Self‑collected         | 50/50                         | 50/50                     | 7,500 / 7,500        |

**Total:** 626,250 training prompts, 86,250 test prompts.

### White‑Box Defence Implementation

The defence works at the model level to explicitly teach the LLM to recognise the boundary of external content.

**Step 1 – Introduce special tokens**  
Add two new tokens to the vocabulary: `<External>` and `</External>` to mark the start and end of external content:

```
P = Combine(T, `<External>` + C + `</External>`, I)
```

**Step 2 – Expand the embedding matrix**  
Concatenate the new token embeddings with the original embedding matrix:

**E_new = Concat(E_origin, E_<External>, E_</External>)**

**Step 3 – Adversarial fine‑tuning**  
Fine‑tune the model on prompts from the BIPIA training set that contain malicious instructions, ensuring the output is unaffected by them. The loss is the standard next‑token prediction cross‑entropy:

**L = – Σᵢ Σⱼ log P(rⱼ⁽ⁱ⁾ | r₁:ⱼ₋₁⁽ⁱ⁾, Pᵢ)**

where *rⱼ⁽ⁱ⁾* is the *j*‑th token of the response to the *i*‑th sample.

---

## Experimental Setup

### Hardware & Software

- **Models evaluated:** 25 LLMs, including closed‑source (GPT‑4, GPT‑3.5‑turbo) and open‑source (Llama2‑Chat‑70B, Vicuna‑33B/13B/7B, WizardLM‑70B/13B, MPT‑30B‑chat, Guanaco‑33B, CodeLlama‑34B, Mistral‑7B, etc.).
- **Black‑box defence tested on:** GPT‑4, GPT‑3.5‑turbo, Vicuna‑13B, Vicuna‑7B. Temperature = 0, max tokens = 2,000.
- **White‑box defence tested on:** Vicuna‑13B and Vicuna‑7B.  
  - Optimiser: AdamW  
  - Epochs: 1, Learning rate: 2e‑5, Batch size: 128, Max length: 2,048  
  - Inference: Temperature = 0, max tokens = 512
- **Evaluation metrics:** ASR determined via rule‑based, LLM‑as‑judge, and language‑detection assessments. Benign performance measured by ROUGE‑1.

### Automated Evaluation Pipeline

A three‑tier pipeline is used:
1. **Rule‑based** – checks if the output follows the injected instruction.  
2. **LLM‑as‑judge** – uses a separate LLM to judge output correctness.  
3. **Language detection** – detects unexpected language shifts (e.g., from English to another language).

---

## In‑Depth Analysis

This paper turns a vaguely perceived security problem into a measurable, systematically solvable engineering challenge.

**The counter‑intuitive finding is thought‑provoking.**  
The fact that stronger models are more vulnerable suggests a fundamental tension between *helpfulness* and *safety* – more capable models are better at following instructions (including malicious ones). This implies that simply scaling up model capacity without accompanying safety mechanisms may widen the attack surface.

**Positional bias matters.**  
The higher ASR when instructions are placed at the end of external content points to a training‑data bias – models learn to expect instructions at the end. This reminds us that safety issues are not just about intent understanding but also about data distribution and attention mechanisms.

**The defence is pragmatically sound.**  
Instead of over‑engineering, the approach directly tackles the root causes: help the model distinguish information from instructions, and explicitly tell it not to follow external commands. The multi‑turn dialogue trick in the black‑box defence is a simple yet effective “sandboxing” method.

**Limitations to consider:**  
- Adaptive attackers could design new prompts to bypass these defences.  
- White‑box defence requires retraining, which may be costly for large models.  
- The benchmark may not cover all real‑world attack variants.

---

## Practical Recommendations

If you are building an LLM application that processes external content, here are actionable takeaways:

1. **Start with the black‑box defence** – for closed‑source APIs, use multi‑turn dialogue + in‑context learning + explicit reminders. This costs almost nothing and provides significant protection.

2. **For high‑risk applications (finance, healthcare, government)** that use open‑source models, invest in the white‑box defence – special tokens + adversarial fine‑tuning. The security gain (ASR ≈ 0) justifies the training cost.

3. **Be mindful of position** – avoid placing external content after the user instruction; put it at the beginning or middle to reduce risk.

4. **Continuously evaluate** – integrate the BIPIA benchmark into your CI/CD pipeline to regularly test against new attack patterns.

---

## References

- Original paper: [Benchmarking and Defending Against Indirect Prompt Injection Attacks on Large Language Models](https://arxiv.org/abs/2312.14197) (arXiv:2312.14197) – *KDD 2025*
- Code repository: [https://github.com/microsoft/BIPIA](https://github.com/microsoft/BIPIA)
