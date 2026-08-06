# Skill-MAS: Evolving Meta-Skill for Automatic Multi-Agent Systems

**Paper**: [arXiv:2606.18837](https://arxiv.org/abs/2606.18837)

---

## TL;DR

Skill-MAS introduces a new paradigm for automatic multi-agent system (MAS) generation. It treats the high-level orchestration capabilities of a meta-agent as an evolvable "meta-skill" — a structured natural-language document that guides task decomposition, agent engineering, and workflow orchestration. Through a closed-loop optimization cycle of **multi-trajectory rollout** and **selective reflection**, Skill-MAS enables frozen frontier LLMs to continuously learn from past experience **without any parameter updates**, breaking the dilemma between model capability and experience retention.

---

## Core Contributions

### Problem Definition

Existing approaches to LLM-based automatic MAS generation fall into two conflicting paradigms:

- **Inference-time MAS** (e.g., EvoAgent, AFlow) rely on frozen large models as meta-agents with iterative search. They leverage strong reasoning but are **experience-agnostic** — they cannot accumulate or transfer diagnostic knowledge across runs.

- **Training-time MAS** (e.g., MAS2, MAS-Orchestra) fine-tune small models (≈7B) to parameterize orchestration. They can internalize experience via gradient updates but suffer from **limited model capacity** and cannot scale to >100B models.

This creates a fundamental dilemma: *either powerful but unlearnable, or learnable but weak*.

### Innovation: The Third Path

Skill-MAS decouples **experience retention** from **parameter updates**. The meta-skill is a structured document comprising three modules:

1. **Task Decomposition** – how to analyze queries, break down subtasks, and define success criteria.
2. **Agent Engineering** – how to instantiate specialized sub-agents with roles and tools.
3. **Workflow Orchestration** – which topology (sequential, hierarchical, or cyclic) and input/output mappings.

The optimization loop:

- **Multi-Trajectory Rollout**: For each task, run K independent trajectories under the current skill. Convert single outcomes into distribution statistics to separate true capability from stochastic noise.
- **Selective Reflection**: Prioritize informative tasks via joint uncertainty–difficulty scoring and adaptive elbow truncation. Apply hierarchical contrastive analysis (intra-task → cross-task) to diagnose systemic failure patterns, then refine the meta-skill based on evidence.

### Key Results

Extensive experiments on four challenging benchmarks (DeepResearchBench, Humanity's Last Exam-Math, BrowseComp-Plus, VitaBench) with four LLMs show:

| Meta-agent | Best Baseline | Skill-MAS-optimized | Gain |
|------------|---------------|----------------------|------|
| Gemini-3.1-Flash | 21.29 (AFlow) | **29.49** | +38.5% |
| DeepSeek-V4-Flash | 35.70 (AFlow) | **41.05** | +15.0% |

- **Cost-performance Pareto**: Training-time methods are cheap but weak; inference-time methods are strong but expensive per sample. Skill-MAS achieves the best trade-off by generating the MAS once (not iteratively per test sample) with moderate upfront optimization cost.

- **Transferability**: Evolved meta-skills transfer strongly across tasks and across LLMs. For example, a skill evolved on BrowseComp-Plus with DeepSeek-V4-Flash improves VitaBench performance by 5.95 points when transferred.

- **Ablation**: Increasing rollout trajectories K (3,5,7) improves performance with diminishing returns.

---

## Technical Details

### Formal Definition of Meta-Skill

The meta-skill `S` is organized into three modules, each covering a phase of the orchestration pipeline:

- **Decomposition**: guidelines for parsing user queries, identifying macro-goals, breaking into cohesive subtasks, and defining evaluable success criteria.
- **Agent Engineering**: rules for instantiating specialized sub-agents with role assignments and tool configurations.
- **Orchestration**: heuristics for choosing topology (sequential, hierarchical, cyclic), defining I/O mappings between agents, and generating executable MAS.

This modular formulation serves two purposes: (i) it provides a principled generation process at inference time, and (ii) it enables precise attribution of failures to specific modules during diagnosis, keeping updates localized and interpretable.

### Multi-Trajectory Rollout

At each round `r`, for each task `t_i` in the validation set, we sample `K` independent trajectories under the current skill `S^(r)`. Each trajectory produces a normalized score `s_{i,k}` ∈ [0,1] and a complete execution trace `Φ_{i,k}`.

We compute two per-task statistics:

- **Uncertainty** (standard deviation across K runs):  
  `u_i = sqrt( (1/K) * Σ (s_{i,k} - mean_i)^2 )`  
  High `u_i` indicates ambiguity or under-specification in the current skill.

- **Difficulty** (negative mean score):  
  `d_i = -mean_i`  
  Harder tasks get higher values.

These turn each task from a pass/fail outcome into a **distributional signature**, allowing the reflection phase to distinguish systemic skill deficiencies from execution noise.

### Selective Reflection

**Priority scoring**: Normalize `u_i` and `d_i` via min-max, then combine:  
`p_i = 0.5*(ũ_i + d̃_i)`

Tasks are sorted by `p_i` descending, and an adaptive elbow truncation selects the most informative ones.

**Hierarchical trajectory reflection**:

- *Intra-task contrast*: Split the K trajectories of each selected task into high-scoring and low-scoring groups. A reflector LLM compares the execution snapshots to identify divergence points, success factors, and recurring failure patterns, producing a per-task summary report.

- *Cross-task synthesis*: Aggregate all per-task reports to extract systemic weaknesses and robust strategies that appear across multiple tasks, forming a structured evidence package `E`.

**Skill update**: An optimizer reviews `E`, removes or rewrites ineffective guidelines, and introduces new rules — all strictly grounded in evidence and generalized as reusable orchestration principles.

---

## Experimental Setup

### Benchmarks

| Benchmark | Domain | Metric |
|-----------|--------|--------|
| DeepResearchBench | Deep research tasks | Comprehensiveness, insight, instruction following, readability |
| Humanity's Last Exam-Math | Expert-level math reasoning | Accuracy |
| BrowseComp-Plus | Complex multi-hop dynamic QA | Accuracy |
| VitaBench | Real-world daily scenarios (multi-tool) | Rule-based success rate |

All metrics normalized to [0, 100%].

### Models Tested

- Proprietary: Gemini-3.1-Flash, GPT-5.4-Nano
- Open-source: Qwen3.5-Plus, DeepSeek-V4-Flash

All components use the same LLM for fair evaluation.

### Baselines

- Inference-time: EvoAgent, AOrchestra, AFlow
- Training-time: MAS2, MAS-Orchestra

### Hyperparameters

- Default trajectories `K=5` (ablation: 3, 7)
- Rounds `R` adaptive based on validation performance
- Final skill `S*` selected by best validation performance for test evaluation

---

## Analysis & Insights

### Theoretical Contribution

Skill-MAS reframes the problem of automatic MAS generation: **experience need not be encoded in weights**. By externalizing orchestration knowledge as a structured natural-language document, we enable frozen LLMs to "learn" — not by changing weights, but by continuously refining the instruction manual that guides their own behavior.

This mirrors how human experts work: a senior architect's value lies not in fixed solutions, but in an evolving *methodology* that guides thinking, decomposition, and design. Skill-MAS makes this methodology explicit and optimizable.

It also resolves a fundamental tension: strong models are too expensive to fine-tune, while fine-tunable models are too weak. Skill-MAS sidesteps this by changing what the model reads, not the model itself.

### Why It Works

Three pillars explain the performance:

1. **Consistent gains across all settings** – even the initial skill (without optimization) often matches strong baselines, showing that a well-designed meta-skill itself is valuable; optimization amplifies this.

2. **Superior cost-performance trade-off** – unlike inference-time methods that search per sample, Skill-MAS optimizes once on a validation set and then generates the MAS in one shot at test time, drastically reducing inference cost.

3. **Strong transferability** – skills evolved on one task/model transfer effectively to others, indicating that the learned strategies are genuinely generic, not overfitted to specific quirks.

### Limitations & Future Work

- **Validation set dependency**: The skill evolution relies entirely on validation feedback. If the validation set is not representative, the skill may overfit. The paper mitigates this via cross-task synthesis and explicit prompts against domain-specific tricks, but the dependency remains.

- **Diminishing returns on K**: Increasing rollout trajectories improves performance but with saturating gains — a practical trade-off between compute and marginal benefit.

- **Dependence on LLM reasoning quality**: The reflector and optimizer themselves rely on the LLM's ability to perform contrastive analysis, root-cause diagnosis, and skill rewriting. If the underlying model has weak reasoning, reflection quality may degrade.

---

## Practical Applications

### When to Use Skill-MAS

- **Continuous improvement without fine-tuning** – ideal for organizations that cannot afford expensive RL or fine-tuning of large models.
- **Cross-scenario reuse** – a skill evolved on one domain can be transferred to another, saving redesign cost.
- **Transparent and interpretable orchestration** – the skill is a plain-text document that teams can read, review, and version-control.

### Implementation Guidelines

1. **Design an initial meta-skill** – follow the three-module structure. Provide clear, general principles rather than task-specific hacks. Quality of the initial skill affects convergence speed and final performance.

2. **Build a representative validation set** – cover diverse difficulty levels and subtask types. This is critical for generalization.

3. **Run iterative evolution** – start with K=3 trajectories, monitor improvement, then increase K if needed. Use the adaptive elbow truncation to save compute.

4. **Evaluate and deploy** – select the best-performing skill on validation, run a final test evaluation, then deploy the skill as a "one-shot" generator for new tasks.

5. **Iterate periodically** – as business scenarios evolve, re-run the evolution loop with fresh validation data to keep the skill up-to-date.

### Cost Considerations

- **Evolution cost**: one-time upfront investment on the validation set (multiple rounds of rollout + reflection). This is analogous to training cost.
- **Inference cost**: very low — one forward pass of the meta-agent with the optimized skill to generate the MAS, no iterative search per test sample.

For most applications, the evolution cost is acceptable, while the low inference cost enables large-scale deployment.

---

## References

- Original paper: Lin, H., Yang, Q., & Qin, C. (2026). Skill-MAS: Evolving Meta-Skill for Automatic Multi-Agent Systems. *arXiv:2606.18837*. [https://arxiv.org/abs/2606.18837](https://arxiv.org/abs/2606.18837)
