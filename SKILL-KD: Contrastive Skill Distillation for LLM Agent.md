
# SKILL-KD: Contrastive Skill Distillation for LLM Agents

## Paper Highlights

SKILL-KD introduces a novel contrastive skill distillation framework that treats skills as explicit distillation media. By performing trajectory contrast between teacher and student agents, it distills behavioral differences into textual "skill patches", then iteratively validates and merges them via drift-aware consolidation to build a stable and reusable skill library. The method significantly outperforms existing skill learning baselines across five mainstream agent benchmarks.

---

## Core Research Content

### Problem Definition

Large language model (LLM) agents rely not only on parametric knowledge but also on reusable procedural patterns when performing multi-step reasoning, tool invocation, and environment interaction. Existing skill acquisition methods typically treat skills as experience summaries, memory entries, or direct abstractions of successful trajectories.

This gives rise to a fundamental **capability gap**: when a weaker "student" agent fails due to lack of task knowledge or operational strategies, its failed trajectory often does not contain sufficient information to infer the missing behaviors. Meanwhile, the teacher's successful trajectory may be too implicit for the student to internalize as actionable guidance. Traditional knowledge distillation relies on parameter updates, but in practice many student models are frozen and cannot be fine-tuned. The core challenge is **how to transfer teacher capabilities to a frozen student agent without updating model weights**.

### Innovative Approach

SKILL-KD redefines skill distillation as an **iterative search-and-verify** process rather than one‑shot summarization. Its technical framework comprises three key mechanisms:

**1. Contrastive Skill Distillation**

Given the same task instance, the student first attempts execution using its current skill library. If it fails, the teacher executes the same task. The system contrasts the student's failed trajectory with the teacher's successful one, and a "consolidation agent" distills the actionable differences into a textual **skill patch**.

Each patch is internally represented as `p = (title, content, why, trace)`, where `title` and `content` form the executable instructions visible to the student, `why` records the rationale for the update, and `trace` links to the source trajectory.

**2. Adaptive Skill Distillation**

Instead of generating skills in one shot, SKILL-KD treats skill extraction as an **iterative search in the textual skill space**. After generating a candidate patch, the system **re‑runs the student agent** to verify its actual effect. If the student still fails, the consolidation agent proposes a revised patch based on the new failed trajectory, and this loop continues until success or reaching a maximum number of rounds.

The core insight: **directly summarizing contrastive differences does not guarantee that the rule will take effect under the student's policy** – the student may interpret instructions differently or require more specific wording to change its behavior.

**3. Drift‑Aware Skill Consolidation**

Repeated local patch updates can cause **skill drift** – patches over‑fit narrow scenarios, add redundant rules, or overwrite previously encoded knowledge. To counter this, SKILL‑KD maintains a **trace‑linked edit history** that records the reason and source trajectory for each modification.

The consolidation agent operates with two tools: a **trace‑link tool** that retrieves full trajectories behind historical edits, and a **patch tool** that performs add, modify, delete, or skip operations. Based on historical evidence, the consolidation agent decides whether to add new rules, delete/modify existing ones, or skip.

### Research Results

The paper evaluates on **five agent benchmarks**:

- **SearchQA** – search‑based question answering
- **SpreadsheetBench** – spreadsheet manipulation
- **DocVQA** – multimodal document QA
- **LiveMath** – mathematical reasoning
- **ALFWorld** – embodied environment interaction

Experiments use **two teacher‑student configurations**:

| Configuration | Student Model | Teacher Model | Type |
|---------------|---------------|---------------|------|
| Group 1 | Qwen3.5‑4B | Qwen3.7‑plus | Homogeneous (same family) |
| Group 2 | Qwen3.6‑35B‑A3B | ChatGPT‑5.5 | Heterogeneous (cross‑family) |

**Main Results** (all scores are hard‑success or exact‑match):

| Method | SearchQA | SpreadsheetBench | DocVQA | LiveMath | ALFWorld | **Average** |
|--------|----------|------------------|--------|----------|----------|-------------|
| **Group 1 (Qwen3.5‑4B → Qwen3.7‑plus)** | | | | | | |
| No Skill | 68.1 | 9.3 | 86.9 | 22.4 | 30.6 | 43.5 |
| EvoSkill | 68.6 | 17.9 | 79.7 | 23.4 | 67.9 | 51.5 |
| Trace2Skill | 68.5 | 19.3 | 88.0 | 27.2 | 64.9 | 53.6 |
| SkillGen | 69.2 | 13.2 | 88.8 | 21.0 | 70.1 | 52.5 |
| SkillOpt | 71.2 | 23.9 | 89.0 | 52.0 | 81.3 | 63.5 |
| **SKILL‑KD** | **79.2** | **24.3** | **89.3** | **54.8** | **86.6** | **66.8** |
| **Group 2 (Qwen3.6‑35B‑A3B → ChatGPT‑5.5)** | | | | | | |
| No Skill | 72.7 | 38.2 | 87.6 | 31.2 | 59.7 | 57.9 |
| EvoSkill | 75.6 | 32.9 | 89.8 | 32.3 | 79.1 | 61.9 |
| Trace2Skill | 75.4 | 33.2 | 90.4 | 29.6 | 70.9 | 59.9 |
| SkillGen | 75.2 | 40.0 | 90.1 | 36.3 | 63.4 | 61.0 |
| SkillOpt | 80.3 | 47.5 | 91.4 | 41.6 | 82.1 | 68.6 |
| **SKILL‑KD** | **80.6** | **49.3** | **92.2** | **54.8** | **96.3** | **74.6** |

**Key Findings**:

- **Cross‑task generalization**: On unseen ALFWorld environments, SKILL‑KD boosted Qwen3.5‑4B from 30.6 to 86.6 (+56.0), proving that skills capture reusable interaction strategies rather than memorised training trajectories.
- **Cross‑family transfer**: Under the heterogeneous setup (Qwen student + ChatGPT teacher), SKILL‑KD still achieved consistent gains, confirming that natural‑language skill carriers do not depend on architectural similarity.
- **Efficiency advantage**: SKILL‑KD reached 66.8 average with only 38 rules (3,010 words), whereas Student‑only required 96 rules (6,821 words) yet only scored 60.1.

### Practical Deployment Potential

SKILL‑KD offers tangible value in several dimensions:

1. **Low‑cost agent enhancement** – No fine‑tuning or retraining; frozen models are improved via an external skill library, drastically reducing deployment costs.
2. **Cross‑model knowledge transfer** – Skills are in natural language, enabling knowledge transfer across different model families and breaking vendor lock‑in.
3. **Interpretable skill inventory** – Each skill includes title, content, and traceability, forming auditable and maintainable intellectual assets.
4. **Continual learning and evolution** – The skill library accumulates and refines over new tasks, while drift‑aware consolidation ensures compactness and stability.

---

## Technical Details

### Core Algorithm Flow

The distillation process of SKILL‑KD can be formalised as follows:

**Input**: training instances D_train, current skill library K, student S, teacher T, consolidation agent C

**For each training instance x**:

1. **Initial student attempt**: τ₀^S = S(x; K), evaluate score r₀^S = r(x, τ₀^S)
2. **If r₀^S = 1**: skip to next instance
3. **If student fails**: run teacher τ_T = T(x; K), get r_T = r(x, τ_T)
4. **Iterative patch generation & verification** (i = 1 to n):
   - Consolidation agent generates patch p_i = C(K, H, (τ₀^S, r₀^S), (τ_T, r_T), E_i)
   - Apply patch and re‑run student: τ_i^S = S(x; Apply(p_i, K))
   - If r(x, τ_i^S) = 1: accept patch, update K, break
   - Otherwise: record failure evidence, continue
5. **If all rounds fail**: no skill update

### Skill Representation and Edit History

The skill library K is a structured Markdown document. Each skill entry contains:

- **title**: skill name
- **content**: actionable instructions (visible to student)
- **why**: rationale (only for consolidation agent)
- **trace**: source trajectory index (only for consolidation agent)

Edit history H is hierarchical: h_j = (op_j, p_j), where op_j ∈ {add, modify, delete, skip}.

### Marginal Gains of Adaptive Rounds

Experiments show that the first adaptive round provides the bulk of improvement, raising average training success from 58.9% to 67.7%. Subsequent rounds still improve but with diminishing returns: second round to 70.8%, third to 72.7%. These extra gains are concentrated on complex tasks requiring multi‑step tool use, such as SpreadsheetBench, LiveMath, and ALFWorld.

---

## Experimental Setup

### Hardware and Model Configuration

- **Student models**: Qwen3.5‑4B (Group 1) and Qwen3.6‑35B‑A3B (Group 2)
- **Teacher models**: Qwen3.7‑plus (Group 1) and ChatGPT‑5.5 (Group 2)
- **Consolidation agent**: same model as the teacher

### Data Splits

- For benchmarks with official splits, native train/validation/test partitions are used.
- For those without, a deterministic 2:1:7 split (random seed 42) is applied.
- The skill library is built **only from training instances**; validation sets are used solely for baseline method selection.

### Evaluation Metrics

Native **hard‑success** or **exact‑match** scores of each benchmark.

---

## Comprehensive Analysis

### Methodological Contribution

The core insight of SKILL‑KD is redefining the essence of “skill” – not as a passive record of experience, but as an **active bridge over the capability gap** between agents. Traditional methods learn from success, while SKILL‑KD learns from **contrast**: the student’s failure reveals “what it lacks”, the teacher’s success shows “what it should have”, and the gap between them is the most valuable training signal.

This resonates with **contrastive case‑based teaching** in human learning – showing both a wrong and a correct case and letting the learner compare the differences is often more effective than showing the correct case alone.

### Comparison with Existing Work

SKILL‑KD systematically compares against four baseline categories:

- **EvoSkill**: iteratively analyses and edits skills from execution failures.
- **Trace2Skill**: consolidates trajectories into a skill catalogue via inductive reasoning.
- **SkillGen**: induces reusable patterns from successful/failed trajectories.
- **SkillOpt**: treats skill documents as optimisable states and edits based on validation feedback.

SKILL‑KD **significantly outperforms all baselines** on all five benchmarks – by 3.3 points on average in Group 1 and 6.0 points in Group 2. This is attributed to two distinctive designs: **using teacher‑student contrast as the core learning signal**, and **framing skill distillation as an adaptive search process**.

### Ablation Insights

**Value of adaptive distillation**:

| Variant | Avg Score | #Rules | #Words |
|---------|-----------|--------|--------|
| No Skill | 43.5 | 0 | 0 |
| Student‑only | 60.1 | 96 | 6,821 |
| Teacher‑only | 58.3 | 80 | 5,683 |
| Batch | 59.4 | 46 | 3,849 |
| Pairwise | 59.0 | 85 | 6,966 |
| **SKILL‑KD** | **66.8** | **38** | **3,010** |

Two key findings emerge:

1. **Teacher trajectories alone are insufficient** – Teacher‑only is the weakest variant, indicating that having strong trajectories does not guarantee student‑usable guidance.
2. **Adaptive verification is critical** – SKILL‑KD uses 38 rules to achieve 66.8, while Student‑only uses 96 rules for only 60.1 – **fewer rules, better results**, proving that the verification‑iteration mechanism filters truly effective skills.

**Value of drift‑aware consolidation**:

Without consolidation, SKILL‑KD drops from 24.3 to 14.6 on SpreadsheetBench and from 54.8 to 27.4 on LiveMath – **performance nearly halves**. This demonstrates that without traceability to historical edits, locally effective patches interfere and override each other, rapidly degrading overall utility.

Case studies reveal three drift patterns:

- **Case‑specific drift**: rules are repeatedly revised with local evidence, causing boundaries to shift.
- **Skill inflation drift**: many individually correct patches encode excessive incidental details (e.g., cell coordinates P6:Q6).
- **Destructive update drift**: new patches overwrite previously useful rules.

### Limitations and Future Directions

1. **Dependence on teacher quality**: Although 38.5% of patches are accepted even when the teacher fails, acceptance rates are significantly higher (45.3%) when the teacher succeeds.
2. **Computational overhead**: Each adaptive round requires re‑running the student; while the first round contributes most gains, additional rounds still incur cost.
3. **Expressiveness ceiling**: Not all procedural knowledge can be fully expressed in natural language rules; complex strategies may require more structured representations.

---

## Practical Applications

### Suitable Scenarios

1. **Model‑as‑a‑Service (MaaS)**: When using third‑party API models that cannot be fine‑tuned, SKILL‑KD offers a way to enhance them via an external skill library.
2. **Multi‑agent collaboration**: In systems with both strong and weak agents, the strong can “teach” the weak through skill distillation.
3. **Domain knowledge codification**: Expert (teacher) strategies can be converted into reusable skill libraries for novice (student) agents, forming organisational intellectual assets.
4. **Cross‑version model migration**: When upgrading to a new model version, accumulated skills from the older version can be transferred, avoiding starting from scratch.

### Implementation Recommendations

1. **Start with single‑round adaptation**: The paper shows the first round yields most benefits; implement single‑round verification first, then expand to multiple rounds as needed.
2. **Maintain edit history diligently**: Drift‑aware consolidation is the key to long‑term stability – ensure full trace‑linking and edit logs.
3. **Set a reasonable iteration cap**: The paper suggests 3 rounds cover most gains; beyond that, marginal returns diminish.
4. **Periodically audit the skill library**: Even with automated consolidation, manual reviews are recommended to ensure rule quality and interpretability.

---

## References

- Original paper: [SKILL-KD: Contrastive Skill Distillation for LLM Agents](https://arxiv.org/abs/2607.28048)
- PDF download: [arXiv PDF](https://arxiv.org/pdf/2607.28048)
- Authors: Qiming Shi, Yibo Dou, Jiawen Zhu, Yulong Tao, Linbo Jin, Zhaolu Kang, Yunfan Zhou, Di Weng (Zhejiang University, Peking University, Alibaba Group)
