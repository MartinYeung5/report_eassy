
# AI4AI at Scale: A Bounded‑Exploration Framework for Scalable Agentic System Optimization

*Analysis of the paper*  
[Original Paper](https://xyz-lab.ai/blogs/ai4ai-at-scale/assets/bounded-exploration-ai4ai-system-optimization.pdf)

---

## Paper Highlights

This work introduces **AI4AI**, a bounded‑exploration, verification‑gated framework that systematically improves the agentic capabilities of large language models. Built on Qwen3.6‑35B‑A3B and Qwen3.5‑397B‑A17B, the resulting systems – XYZ‑Aquila‑mini and XYZ‑Aquila‑pro – achieve state‑of‑the‑art results across multiple benchmarks including BrowseComp, BrowseComp‑ZH, LiveBrowseComp, and WideSearch, outperforming all open‑source models of comparable size and even several closed‑source systems with >1T parameters.

---

## Core Contributions

### Problem Definition

Enhancing LLM agentic capabilities is a complex systems problem. A capable agent must handle long‑term planning, tool use, evidence verification, failure recovery, constraint following, and more – yet these abilities emerge not from a single model checkpoint, but from **tightly coupled components**: model selection, data construction, post‑training strategies, tool interfaces, verifier design, context management, and infrastructure. Traditional approaches that optimise each component in isolation often yield fragile gains, hidden performance regressions, or fail to transfer outside the development environment. Moreover, asking current LLMs to autonomously perform end‑to‑end system optimisation remains far beyond their reliable capability – the search space is huge, component interactions are subtle, and naive automation can hide the true cause of changes, overfit to available feedback, or blur accountability for unsafe decisions.

### Innovative Method

The paper proposes a **bounded‑exploration AI4AI optimisation mechanism** with the following key designs:

1. **Human‑defined optimisation contract** – Humans specify target capabilities, private proxy benchmarks, resource and tool boundaries, risk limits, and acceptance criteria, forming a versioned contract H(νt).
2. **Four‑stage cycle** – An AI agent iteratively executes **Research → Development → Review → Record** under the contract constraints.
3. **Isolated evaluator and acceptance gating** – A separate evaluator E_gate returns metrics and diagnostic signals, but **never exposes answer keys or private labels**; the acceptance policy G decides to adopt or reject based on evidence.
4. **Shared experience memory** – Every attempt – whether accepted or rejected – is fully recorded in a persistent shared memory F_t, including provenance, applicability conditions, confidence scores, etc., so that future cycles can avoid repeated failures and reuse successful experiences.
5. **Human‑AI collaboration** – Humans exercise authority through the contract and exception reviews, while AI handles the operational work – not requiring human approval for every low‑level step.

### Results

XYZ‑Aquila achieves top results across multiple authoritative agent benchmarks:

| Benchmark | XYZ‑Aquila‑mini (<40B) | XYZ‑Aquila‑pro (<400B) |
|-----------|------------------------|------------------------|
| BrowseComp | 78.8% (1st in class) | 84.8% (1st in class) |
| BrowseComp‑ZH | 82.9% (1st in class) | 85.1% (1st in class) |
| LiveBrowseComp | 48.7% (1st in class) | 53.7% (1st in class) |
| WideSearch | 80.8% (1st in class) | 81.2% (1st in class) |
| DeepSearchQA | 89.5% (1st in class) | 92.5% (2nd in class) |
| Humanity's Last Exam | 51.1% (1st in class) | 53.3% (1st in class) |

Notably, XYZ‑Aquila‑pro **sets new state‑of‑the‑art records** on BrowseComp‑ZH, LiveBrowseComp, and WideSearch among all publicly reported systems (including closed‑source models with over 1T parameters).

### Practical Deployment Potential

The framework is broadly applicable – the paper emphasises that the AI4AI optimisation mechanism is not limited to Deep Search but can be generalised to **any complex AI system development scenario**, including:
- **LLM post‑training pipeline optimisation** – automatically diagnosing and iteratively fixing flaws in SFT/RL/DPO strategies.
- **RAG system tuning** – optimising retrieval strategies, context windows, and tool‑call configurations.
- **Software engineering agents** – assisting in code generation, testing, and debugging iterations.
- **Scientific automation** – similar to the AI Scientist concept, but with added bounded constraints and auditability.

---

## Technical Details

### Formal Definition of the Optimisation Contract

The agentic system is formalised as a configuration θ covering seven dimensions: model, learning, policy, tools, data, development evaluation, and infrastructure:

```
θ ∈ Θ,   Θ ⊆ Θ_model × Θ_learn × Θ_policy × Θ_tool × Θ_data × Θ_dev‑eval × Θ_infra
```

The human‑defined versioned optimisation contract is:

```
H(ν_t) = ⟨J, B_priv, E_gate, C_H, G, B_cycle, τ_stop⟩
```

Where:
- **J** – target capabilities
- **B_priv** – private proxy benchmark
- **E_gate** – isolated evaluator
- **C_H** – admissibility predicate
- **G** – versioned acceptance policy
- **B_cycle** – per‑cycle resource budget
- **τ_stop** – stopping condition

### Core Formulas of the Four‑Stage Cycle

**Research** – The research module diagnoses the current system and proposes admissible interventions:

```
(a_t, û_t, r_t) = R_φ(θ_t, F_t, h_t; H(ν_t))
a_t ∈ A_H(θ_t),   c(a_t) ≤ B_cycle
```

**Development** – The intervention is applied to generate a candidate system:

```
θ̃_t = f(θ_t, a_t),   C_H(θ̃_t) = 1
```

**Review** – The isolated evaluator produces an evidence package:

```
z_t = E_gate(A_θ̃_t; B_priv)
d_t = G(z_t, z_ref_t, a_t; H(ν_t)) ∈ {accept, reject, escalate}
```

**Record** – The full cycle evidence is recorded and the shared memory updated:

```
e_t = ⟨θ_t, a_t, θ̃_t, z_t, d_t, r_t, ν_t, v_t⟩
F_{t+1} = Update(F_t, e_t)
```

### Five Dimensions of Boundedness

“Bounded” does not mean a small search space, but **constrained permissions, evidence access, and resource usage**:

1. **Scope boundary** – Only specified design surfaces and intervention categories may be modified.
2. **Authority boundary** – Only authorised data sources, tools, models, and infrastructure may be used.
3. **Evaluation boundary** – Hidden labels, answer keys, and external benchmarks are invisible to the optimiser.
4. **Resource boundary** – Per‑cycle limits on compute, time, rollouts, and evaluation budgets.
5. **Escalation boundary** – High‑risk, ambiguous, or out‑of‑contract cases must be rejected or escalated.

---

## Experimental Setup

### Hardware & Software Configuration

XYZ‑Aquila series are built on:

| Component | XYZ‑Aquila‑mini | XYZ‑Aquila‑pro |
|-----------|----------------|----------------|
| Base model | Qwen3.6‑35B‑A3B | Qwen3.5‑397B‑A17B |
| Parameter scale | < 40B | < 400B |
| Architecture | MoE (Mixture of Experts) | MoE |

### System Design Highlights

- **Graph grounding + tool‑reachable task construction** – agents plan and gather evidence based on graph structures.
- **State‑preserving SFT** – supervised fine‑tuning maintains state consistency.
- **Fixed three‑tool execution contract** – explicit context policies and precise state replay.
- **External evaluation isolation** – external benchmarks are not used in routine optimisation cycles; they are reserved for final or explicitly authorised evaluations.
- **Low‑cost screening** – for expensive intervention types (e.g., RL), low‑cost pilot validations are run first; full training and evaluation budgets are only committed after passing pilots.

### Open Resources

The team has released model weights, evaluation code, training details, representative trajectories, and versioned intervention records:
- **ModelScope**: https://www.modelscope.cn/models/XYZAILab
- **Hugging Face**: https://huggingface.co/XYZAILab
- **GitHub**: https://github.com/XYZ-AI-Lab

---

## Comprehensive Analysis

### Theoretical Contributions

The core theoretical advance lies in **transforming AI system optimisation from a “black‑box tuning” exercise into an auditable, reproducible governance process**. Traditional AutoML or hyperparameter optimisation often focuses on single numeric dimensions, whereas the AI4AI framework expands the optimisation object to the entire system design space – from models, data, and training strategies to tools and infrastructure – and introduces a human‑defined contract as a “constitution” that bounds AI exploration.

This paradigm shift carries deep academic significance. The paper explicitly argues that agentic capabilities are **emergent properties of the complete system**, not of an isolated model checkpoint. This challenges the current model‑centric evaluation paradigm and offers a new theoretical lens for agentic AI assessment and optimisation.

### Practical Value

From a practitioner’s perspective, AI4AI addresses three core pain points:

1. **Interpretability** – Every intervention is fully recorded, including rejected attempts and their failure reasons. Optimisation becomes a traceable engineering process, not alchemy.
2. **Overfitting prevention** – Hiding private labels, isolating external benchmarks, and limiting per‑cycle evaluation counts effectively guard against over‑adaptation to the development set.
3. **Balanced human‑AI collaboration** – The framework neither relies on tedious manual tuning (inefficient) nor on fully autonomous AI optimisation (dangerous), but achieves **controllable automation** through contracts, gating, and exception escalation.

### Limitations and Future Work

The paper candidly acknowledges several limitations:

- Benchmark comparisons are based on heterogeneous public reports; different systems may use different evaluation tools, judges, or timestamps – not a strictly controlled total ordering.
- The framework’s effectiveness has so far only been verified on the Deep Search scenario; generalisation to other AI systems (e.g., multimodal, embodied agents) requires further research.
- While bounded exploration enhances safety, it may also limit certain “out‑of‑the‑box” innovative discoveries.

---

## Practical Recommendations

### For Researchers

1. **Start small** – If adopting AI4AI in your own projects, begin with a single optimisation dimension (e.g., only SFT data or only tool‑calling strategies) and gradually expand to multi‑dimensional joint optimisation.
2. **Value failed records** – One of the most inspiring designs is the **complete recording of rejected attempts**. In practice, building a “failure case library” is often more valuable than only logging successes – it helps subsequent cycles avoid repeating mistakes.
3. **Design contracts carefully** – The risk boundaries and acceptance criteria are the foundation of safety. Start with stricter constraints and relax them as understanding of the system deepens.

### For Engineering Teams

1. **Build shared experience memory** – The Shared Experience Memory is essentially a digital deposit of team knowledge. Engineering teams can adopt this idea by structuring all experiments (successes and failures) – their context, configurations, results, and human reviews – into a persistent store, forming a “collective intelligence” for the organisation.
2. **Separate evaluation from optimisation** – The “isolated evaluator” reminds us to strictly distinguish **signals used to guide optimisation** from **signals used for final assessment** – the former can be used frequently, while the latter must be carefully preserved.
3. **Low‑cost pilots first** – For expensive intervention plans (e.g., large‑scale RL training), conducting low‑cost pilots first is a wise engineering practice.

---

## References

- Original Paper: https://xyz-lab.ai/blogs/ai4ai-at-scale/assets/bounded-exploration-ai4ai-system-optimization.pdf
- Models (ModelScope): https://www.modelscope.cn/models/XYZAILab
- Models (Hugging Face): https://huggingface.co/XYZAILab
- Datasets (ModelScope): https://www.modelscope.cn/datasets/XYZAILab
- Code Repository: https://github.com/XYZ-AI-Lab
