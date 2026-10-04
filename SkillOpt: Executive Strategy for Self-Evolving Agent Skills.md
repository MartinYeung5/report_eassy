# Training Markdown as Weights: How Microsoft SkillOpt Lets Agent Skills Evolve on Their Own

## Paper Highlights

SkillOpt is the first systematic and controllable text-space optimizer proposed by Microsoft Research in collaboration with Shanghai Jiao Tong University, Tongji University, and Fudan University. Its core idea is to treat an Agent's natural language skill document as "trainable parameters" and iteratively optimize it using a training loop similar to deep learning. Across all 52 evaluation units spanning 6 benchmarks, 7 target models, and 3 execution frameworks, SkillOpt achieved optimal or tied-optimal performance. On GPT-5.5, it improved the accuracy of the no-skill baseline by 19.1 to 24.8 percentage points.

## Core Research Content

### Problem Definition

Today's Agent skills—those CLAUDE.md files, Codex skill files, and various system prompts—are almost entirely hand-written or generated once by an LLM. You write a version, run a few tasks to see how it performs, tweak it where it feels off, and repeat. This process is no different in essence from manually tuning a prompt, except the object has changed from a single sentence to an entire document.

The root of the problem is that skill editing lacks a reproducible, feedback-driven, and convergent optimization paradigm like that of a deep learning optimizer. We don't know whether a change makes things universally better or simply robs Peter to pay Paul.

Meanwhile, adapting model weights is entirely unavailable for closed-source frontier models and prohibitively expensive for open-source ones. Skill documents happen to provide a natural adaptation interface—they encapsulate program steps, domain heuristics, tool policies, output constraints, and failure modes, allowing a frozen model to adapt through external text.

### Innovative Method

The core insight of SkillOpt can be summed up in one sentence: **An Agent's skill document is its "external weights." Since internal weights can be optimized with gradient descent, external weights should also have a systematic training method.**

It builds a four-step training loop that is structurally almost isomorphic to a deep learning training loop:

**Rollout (Forward Pass)**: The frozen target model executes a batch of tasks with the current version of the skill document, recording complete execution trajectories—messages, tool calls, verification feedback, and final scores. This step produces "evidence," equivalent to the forward pass of a neural network.

**Reflect (Backward Pass)**: An independent optimizer model analyzes these trajectories. The key design is that failure cases and success cases are reflected upon separately. Failure minibatches are used to discover which operational rules need correction, while success minibatches are used to confirm which existing rules are working and must not be touched. Failure-driven edit proposals are given higher merging priority.

**Edit (Parameter Update)**: The optimizer proposes structured add/delete/replace edit operations, but with a strict budget on the number of edits—this is the "text learning rate."

**Gate (Validation Gate)**: The candidate new skill must be run on a held-out validation set, and is accepted only if performance strictly improves. This gate turns "reflection" into propose-and-test optimization, rather than unconditional self-editing.

### Research Results

The experimental results are rigorous. In GPT-5.5 direct chat mode: SearchQA improved from 77.7 to 87.3, SpreadsheetBench jumped from 41.8 to 80.7, OfficeQA improved from 33.1 to 72.1, DocVQA improved from 78.8 to 91.2, LiveMath improved from 37.6 to 66.9, and ALFWorld improved from 83.6 to 95.5.

Among these, SpreadsheetBench nearly doubled (+38.9 points), and OfficeQA went from 33.1 to 72.1 (+39 points). These two benchmarks are precisely the scenarios with the highest demand for "procedural capability"—multi-turn tool calls, code generation, and real openpyxl/pandas runtime.

The transfer experiments are equally noteworthy: a spreadsheet skill trained on GPT-5.5 with Codex still brought positive gains when transferred to Claude Code; a skill optimized on OlympiadBench remained effective when transferred to Omni-MATH. This means skills can be "trained once, audited as text, and reused across models, frameworks, and tasks."

### Potential for Practical Deployment

The deployment path is very clear: after training, what is exported is a single best_skill.md file, typically 300–2000 tokens. At deployment time, the model and inference pipeline remain completely unchanged; only the originally hand-written skill file is replaced with a validated version. **There are zero additional model calls and zero latency increase during inference.**

Currently, SkillOpt has earned over 12,000 stars on GitHub, is open-sourced under the MIT license, and can be installed via `pip install skillopt`. Version v0.2.0 also adds SkillOpt-Sleep—an overnight offline self-evolution engine that allows local coding Agents to automatically review sessions, replay tasks, and consolidate experience during idle time.

## Technical Details

### Text Learning Rate: Preventing Catastrophic Forgetting

In deep learning, too high a learning rate leads to catastrophic forgetting. In text space, SkillOpt encounters exactly the same problem: if an edit changes too much at once, it may overwrite previously learned effective rules.

The solution is to limit the number of edit operations allowed per step. The paper's default setting is `lr=4`, meaning at most 4 add/delete/replace operations per step. Ablation studies confirm the necessity: removing the learning rate constraint drops SearchQA performance from 87.1% to 84.6%, SpreadsheetBench from 77.5% to 75.7%, and LiveMath from 61.3% to 57.3%.

SkillOpt supports four scheduling strategies: constant, linear, cosine, and autonomous, with cosine decay as the default—larger steps for exploration early on, smaller steps for consolidation later.

### Rejected Edit Buffer: Negative Feedback Memory

When an edit proposal is rejected by the validation gate, it is not simply discarded. Instead, it enters an epoch-local buffer that records the failure pattern and the corresponding performance drop. Subsequent reflection calls within the same epoch receive this buffer, avoiding repeated attempts at already-failed edit directions. This gives the entire training loop a negative feedback mechanism without adding any inference-time cost.

### Slow Update and Meta-Skill: Cross-Epoch Memory

At the end of each epoch, the system samples tasks from both the previous and current epochs for comparison, retaining edit directions that hold consistently across epochs. This plays a role similar to a momentum term. Meanwhile, a "meta-skill" summarizes accepted and rejected patterns, serving as the optimizer's cross-epoch strategy memory.

### Deep Theory: Why "Gradient Descent" in Text Space Works

The most thought-provoking aspect of SkillOpt is that it reveals a deep structure: **the optimization of natural language instructions follows the same mathematical and structural patterns as neural network training.** This is not a metaphor-level analogy, but an operational isomorphism.

Both weight space and text space are high-dimensional parameter spaces with redundant representations. In weight space, gradient descent finds generalization directions by aggregating gradient signals from many samples; in text space, SkillOpt finds procedural correction directions by aggregating reflection signals from many trajectories. The commonality is that **the signal from a single sample is noise, while the aggregated signal is the gradient.**

This is most fully embodied in SkillOpt's minibatch reflection design. A single trajectory often produces "anecdotal fixes"—patching a specific case. But a minibatch exposes reusable procedural errors: the Agent systematically searches the wrong source, systematically writes the wrong output format, systematically fails to verify tool results. These are where the "gradient" lies.

Another noteworthy deep mechanism is that **the validation gate is not merely an engineering safety measure; it essentially defines a "monotonic loss decrease" property in text-space optimization.** In weight space, learning rate scheduling and gradient clipping together ensure the loss does not diverge; in text space, the text learning rate (edit budget) ensures each change is not too large, and the validation gate ensures each accepted direction is correct. Together, they give the optimization process a monotonic improvement guarantee similar to that in deep learning.

This also explains why "unconstrained rewriting" fails. When the optimizer can freely rewrite the entire skill document, it is effectively jumping around a huge, non-convex, discontinuous search space, easily leaping from one local optimum to a worse one. Bounded editing essentially imposes a "trust region" constraint in text space—searching for improvement only within a small neighborhood at each step.

### Case Study: What a Learned Skill Looks Like

Taking SpreadsheetBench as an example, the skill optimized by SkillOpt is procedural rather than instance-specific. The rules it learns are like: "Read the original cell format before modifying formulas," "Check openpyxl's merged_cells attribute before handling merged cells," "Verify that the DataFrame shape matches expectations before outputting." These are reusable operational constraints, not hard-coded for a specific spreadsheet.

## Research Setup

### Hardware and Software Configuration

SkillOpt itself is a pure Python framework requiring Python 3.10+. During training, it needs to call APIs of the target model and optimizer model, so the main "hardware" cost actually comes from API call expenses rather than local GPUs.

The training pipeline requires three data splits: `train/` for generating execution trajectories, `selection/` for the validation gate, and `test/` for final evaluation. Data is organized as split directories.

Key hyperparameter settings are as follows:

- **Epochs**: 4 (skills converge much faster than neural networks; 2–4 epochs are usually sufficient)
- **Rollout batch size**: 40 tasks per step
- **Reflection minibatch size**: 8, with 8 parallel analysis workers
- **Text learning rate**: 4, with cosine decay (lower bound 2)
- **Validation gate**: Accept only if strictly greater than the current selection score; ties are also rejected
- **Slow update**: Sample 2 tasks per epoch to compare skill performance before and after
- **Edit mode**: Default patch mode (local add/delete/replace), alternative is rewrite_from_suggestions

### Execution Framework Adaptation

SkillOpt is validated under three execution modes:

1. **Direct chat**: The skill is prepended directly to the system prompt, with a single chat completion
2. **Codex harness**: Driven via the codex CLI in a workspace-write sandbox, with the skill rendered as a per-task SKILL.md
3. **Claude Code harness**: Same workspace contract as Codex, implemented via Claude Code

An important engineering detail: under tool-using frameworks, the optimizer does not just look at the final answer; it also reads a compact execution trace summary (`codex_trace_summary.txt`) to understand what the Agent actually did, not just what it said.

## Comprehensive Analysis

### Where the Real Contribution Lies

The value of SkillOpt does not lie in any single technical component—bounded editing, validation gates, and reflection loops all have precedents in the prompt optimization literature. Its real contribution is **elevating skill optimization from an "engineering trick" to a "training method."**

Before this, the skill editing process was: find a problem → manually modify → verify by trial and error. SkillOpt turns it into: sample evidence → aggregate reflections → bounded updates → strict validation → retain negative feedback → cross-epoch memory. Every step of this pipeline has a clear design rationale and ablation evidence, rather than being decided arbitrarily.

More notably, it repositions the concept of "adaptation." In the current Agent technology stack, model weights are the alignment layer, prompts are the interaction layer, and skills are the procedural layer. SkillOpt precisely identifies the procedural layer as the bottleneck for domain adaptation—the model knows how to reason, but not your codebase's conventions, your API's pitfalls, or your document format's specifications. This knowledge can now be systematically trained into a text file.

### Deep Logic Behind Design Decisions

Several design decisions look simple but are actually deeply considered.

**Why separate failure and success reflections?** If mixed together for the optimizer to analyze, it easily falls into a "compromise" trap—seeing success makes it inclined to preserve the status quo, while seeing failure makes it inclined to make big changes. After separation, the failure minibatch focuses on finding problems, and the success minibatch focuses on confirming effective rules. When merging, failure corrections take priority. This is an information filtering mechanism that prevents successful "noise" from diluting the "signal" of failure.

**Why use an independent optimizer model instead of letting the target model reflect on itself?** Letting the target model modify its own skill creates a fundamental conflict of interest: it tends to believe its own approach is correct. Using a stronger frontier model as an independent optimizer introduces a "teacher" role that can see the target model's systematic problems from an external perspective. Moreover, this optimizer is only used during training and is completely unnecessary at deployment.

**Why can slow updates coexist with fast updates?** The per-step edit is "fast" and local; the per-epoch slow update is "slow" and global. Fast updates respond to specific failure patterns, while slow updates identify stable improvement directions across epochs. Separating the two prevents local corrections from being mistaken for global trends.

### Positioning Relative to Related Work

Compared with GEPA (reflective prompt evolution), SkillOpt's optimization object is the complete skill document rather than a single prompt, and it introduces stricter training controls. Compared with Trace2Skill (trajectory-to-skill distillation), SkillOpt does not extract skills from trajectories in one shot but iteratively trains them. Compared with skill evolution methods like EvoSkill, SkillOpt's core difference is the **validation gate**—not all "seemingly reasonable" edits are accepted, but only those that truly improve performance on held-out data.

### Limitations

Several limitations of the paper are worth noting. First, the training process requires calling frontier model APIs, which may not be cheap for large-scale training scenarios. Second, skill convergence speed varies by task—some benchmarks may need more epochs, but the paper does not deeply discuss theoretical guarantees for convergence. Third, the "strictly greater than" strategy of the validation gate avoids overfitting, but in some cases may be too conservative, causing useful small improvements to be rejected. Finally, the paper is mainly validated on English benchmarks, and its applicability to multilingual scenarios has not yet been explored.

## Practical Applications

### When to Use SkillOpt

SkillOpt is best suited for **repetitive tasks with clear evaluation metrics**. If your Agent needs to repeatedly handle a certain type of task (spreadsheet analysis, document QA, domain-specific retrieval augmentation) and you have a way to automatically score it (answer matching, verifier feedback, a small manually labeled validation set), SkillOpt can be useful.

Scenarios that are less suitable: tasks whose evaluation is entirely subjective (e.g., creative writing), or tasks so variable that no "reusable procedural knowledge" exists.

### Practical Advice

**Start with a "good enough" initial skill.** SkillOpt trains from an initial skill; too poor a starting point will slow down the optimization process. First hand-write a version that works, even if rough, and let SkillOpt iterate on it.

**Design the validation set carefully.** The validation gate is the gatekeeper of the entire system. If the validation set is too small or biased, the gate will let through edits that happen to improve on the validation set but are actually harmful. It is recommended to have at least 100–200 samples in the validation set, with a distribution consistent with the training set.

**Monitor the patterns of rejected edits.** The rejected edit buffer is not just negative feedback during training; it is also a diagnostic signal. If you find many edits rejected, it may indicate that the current skill is already near a local optimum, or that the validation set has issues. If almost no edits are rejected, it may indicate the learning rate is too low or the optimizer is not aggressive enough.

**Consider the cost-benefit of cross-framework transfer.** The paper validates that skills can transfer across execution frameworks, but the transfer is not lossless. If your deployment environment differs from the training environment, it is recommended to do a lightweight fine-tuning round (a few epochs) on the target framework before deployment.

### Long-Term Value of SkillOpt from an Engineering Perspective

SkillOpt represents a broader trend: **structured text instructions are becoming first-class optimization targets.** Together with Anthropic's CLAUDE.md and VS Code's .instructions.md, this trend is already clear. In the future, teams may no longer "write prompts" but "train prompts"—provide data, provide evaluation, and let the optimizer automatically find the best instructions.

For Agent developers, this means the focus of skill engineering will shift from "how to write" to "how to evaluate" and "how to design training data." The ability to write a good skill may be less important than the ability to design a good evaluation system. This shift is not easy for many people, but it is happening.

## References

- Original Paper: [SkillOpt: Executive Strategy for Self-Evolving Agent Skills](https://arxiv.org/abs/2605.23904)
- Open Source Code: [github.com/microsoft/SkillOpt](https://github.com/microsoft/SkillOpt)
- Project Website: [microsoft.github.io/SkillOpt](https://microsoft.github.io/SkillOpt/#idea)
- Deep Learning Analogy Doc: [DL ↔ SkillOpt Analogy](https://github.com/microsoft/SkillOpt/blob/main/docs/guide/dl-analogy.md)
