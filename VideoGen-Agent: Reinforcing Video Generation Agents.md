VideoGen-Agent: Reinforcing Video Generation Agents# VideoGen-Agent: Reinforcing Video Generation Agents

> A multimodal agent trained with multi-task reinforcement learning to orchestrate enhancement, generation, and verification tools for video generation.

## Highlights

VideoGen-Agent introduces a multimodal agent trained with multi-task reinforcement learning that coordinates enhancement, generation, and verification tools to complete video generation tasks. The work performs two-stage training (supervised fine-tuning + reinforcement learning) on a class-balanced dataset covering six tasks, and builds VABench, a benchmark with 600 prompts. Key results show that VideoGen-Agent improves over the base text-to-video model by 19.1 points, and upgrading the generation tools without retraining the agent further pushes the score to 86.1.

## Core Research Content

### Problem Definition

Although current video generation models have made significant progress in fidelity and temporal coherence, they still struggle with prompts that require specialized knowledge, specific identity preservation, physical consistency, or ordered events. More specifically, single-pass generation tools have four structural weaknesses: **uncontrollable** (characters deform and scenes drift in long videos), **inconsistent** (characters and lighting lack consistency across shot changes), **non-batchable** (dozens of videos in commercial projects cannot be quality-managed uniformly), and **non-auditable** (output quality depends on manual frame-by-frame inspection and cannot be integrated into automated pipelines).

The core judgment of the paper is that the problem is not that the generation models themselves are not strong enough, but that there is a missing intelligence layer that can plan, execute, inspect, and repair. VideoGen-Agent does not try to find a smarter generation model. Instead, it builds an agentic management shell around existing models that can actually do the work, allowing occasional quality fluctuations of the base model to be absorbed by a procedural verification and repair mechanism.

### Novel Method

**First, a three-layer architecture for tool orchestration.** The agent coordinates three types of tools through multi-turn interaction: enhancement tools (e.g., prompt rewriting, reference image retrieval), generation tools (covering T2V, I2V, R2V, M2V, and other conditional generation modes), and verification tools (which assess and provide feedback on generated video quality). Since no single generation model can simultaneously satisfy all conditional input, generation speed, and quality balance requirements, the paper configures separate generation tools and sets up two complete toolsets: Toolset 1 for training, and Toolset 2, which upgrades all generation tools while keeping the same enhancement and verification tools.

**Second, a two-stage training pipeline.** Using Qwen3-VL-8B-Instruct as the base model, the model first undergoes supervised fine-tuning (SFT) on teacher-generated trajectories to establish basic tool-calling behavior. Then, GRPO (Group Relative Policy Optimization) is used for reinforcement learning optimization, with a category-aware hybrid reward function that jointly evaluates the effectiveness of tool calls, task-appropriate tool usage, and the quality of generated videos. This "imitate first, then explore" strategy allows the agent to surpass the performance ceiling of the initial SFT stage.

**Third, the VABench benchmark.** The paper constructs a held-out benchmark with 600 prompts, 100 per task category, covering six dimensions: procedural knowledge, single-entity identity preservation, multi-entity identity preservation, physical consistency, scene composition, and multi-shot temporal structure. Each prompt is human-reviewed to ensure it is suitable for visual presentation and appropriately difficult.

### Results

On VABench, VideoGen-Agent with Toolset 1 achieves an overall score of 75.6, improving over its base T2V generator (Seedance 1.0) by 19.1 points (from 56.5 to 75.6), and surpassing the strongest standalone baseline Seedance 2.0 by 2.4 points. Under the Toolset 2 configuration, the score further rises to 86.1, achieving the highest score in all six task categories, and this improvement requires **no additional agent training**—meaning the agent's learned "tool-use ability" generalizes across tools and can directly benefit from iterative progress in the generation tools themselves.

In human evaluation, evaluators preferred the outputs from the upgraded configuration over the strongest standalone baseline in 84.3% of comparisons. Ablation studies further confirm the contribution of each training stage: from zero-shot tool use to SFT, and then to full RL training, performance increases monotonically, validating the necessity of the two-stage training design.

### Practical Applicability

The practical value of this work lies in two main directions. First, **automation of video generation workflows.** The closed-loop architecture demonstrated in the paper (understand intent → plan shots → generate clips → quality check → repair and retry → confirm and archive) can be directly transferred to scenarios such as batch short-video production, e-commerce product videos, and educational content creation. Second, **compatibility with the tool ecosystem.** The tool-use policy learned by the agent is not tied to a specific generation model. When the underlying generation tools are upgraded, the agent can benefit without retraining—which, for the rapidly iterating field of AI video generation, means lower maintenance costs and stronger technical adaptability.

## Technical Details

### Agent Architecture

The overall architecture of VideoGen-Agent can be broken down into four key modules:

- **Planner**: Converts the user's natural language description into a structured shot list. Each shot uses a fixed-field format (scene description, shot type, duration, reference elements, etc.) to avoid uncontrollability caused by free-form generation.
- **Executor**: Calls the appropriate enhancement and generation tools according to the shot list, completing the generation from prompt to video clip.
- **Checker**: Performs multi-layer quality verification on the generated videos, including format correctness, tool-call validity, and video quality scoring based on a VLM (Gemini 3.1 Pro).
- **Memory**: Records the inputs, outputs, and historical trajectories of each tool call, providing context for subsequent repair decisions.

### Reward Function Design

The paper adopts a category-aware hybrid reward composed of three signals:

$$
R = \lambda_1 \cdot R_{\text{format}} + \lambda_2 \cdot R_{\text{tool}} + \lambda_3 \cdot R_{\text{video}}
$$

where \(R_{\text{format}}\) evaluates the format correctness of tool calls, \(R_{\text{tool}}\) measures how well the tool selection matches the task type, and \(R_{\text{video}}\) is the VLM-judged quality score of the generated video. Different task categories use different weight coefficients to adapt to their respective quality evaluation priorities.

### Training Configuration

The SFT stage uses 7K high-quality trajectories generated by the teacher model, including complete reasoning chains, tool-call records, intermediate observations, and final generated videos. The reinforcement learning stage is conducted on a compute cluster of four NVIDIA H800 (80GB) GPU nodes, using the GRPO algorithm for multi-task policy optimization. Notably, training uses Toolset 1 throughout, while Toolset 2 is only used during evaluation to test the agent's generalization ability.

## Research Setup

The baseline comparison covers both open-source and closed-source camps. Open-source baselines include CogVideoX-5B, Mochi-1, HunyuanVideo-13B, and Wan2.1-T2V-14B; closed-source baselines include Hailuo 2.0, Kling 3.0, Seedance 1.0 Pro Fast, and Seedance 2.0 Fast. All baselines run in single-pass generation mode, i.e., without any external tools or iterative optimization.

The ablation study is designed rigorously: under the fixed Toolset 1 condition, it sequentially compares the original T2V generator, prompt rewriting, zero-shot tool use, the SFT model, and the full RL-trained model, clearly isolating the independent contribution of each training stage.

## Comprehensive Analysis

The real value of this work is not the improvement of any single technical metric, but that it proposes a **composable and evolvable** paradigm for video generation. The traditional approach is to wait for a stronger end-to-end generation model to solve all problems. VideoGen-Agent instead decouples "generation" from "decision-making": the generation model is responsible for drawing, while the agent is responsible for judging whether the drawing is correct, whether to switch tools, and whether to regenerate. The direct benefit of this separation is that advances in generation tools can be translated into overall system capability improvements in a plug-and-play manner—in the Toolset 2 experiments, the agent had never seen the new tools but still used the upgraded tools correctly and achieved significant improvements, demonstrating the generalization ability of the tool-use policy.

Of course, there is room for deeper discussion. The current system still falls below Seedance 2.0 in the "multi-entity identity preservation" and "scene composition" categories (even with Toolset 2, the paper acknowledges that it "remains below this model" in some categories), indicating that the advantages of tool orchestration cannot fully compensate for the shortcomings of base generation capability in certain dimensions. In addition, the latency and quality ceiling of the entire pipeline are constrained by the quality and speed of the generation tools themselves, and the feedback granularity of the verification module has room for further refinement.

## Practical Applications

For teams looking to introduce similar solutions in real projects, the following points are worth considering:

**Build evaluation before building the pipeline.** The success of VideoGen-Agent is largely built on VABench, a structured evaluation benchmark. Without clear evaluation dimensions (six categories) and quantifiable scoring criteria, the effectiveness of tool orchestration cannot be measured or optimized. In real projects, it is advisable to first define "what counts as a qualified video" before designing the agent's decision logic.

**SFT is a necessary cold start.** The ablation results for zero-shot tool use clearly show that an untrained model cannot effectively use tools even with a complete toolset and system prompt. Supervised fine-tuning on teacher trajectories establishes basic tool-use intuition for the agent, which is a prerequisite for subsequent reinforcement learning optimization.

**The compounding effect of tool upgrades deserves attention.** The most impressive finding of the paper is that the tool-use ability learned by the agent can be directly transferred to upgraded tools not seen during training. This means that in engineering practice, "agent capability" and "generation tool capability" can be managed as two independent iteration dimensions, and their progress can be additive rather than mutually bound.

## References

- Original Paper: https://andyca111.github.io/VideoGen_Agent/
- arXiv: https://arxiv.org/abs/2609.24997
- GitHub Project: https://github.com/AndyCA111/VideoGen_Agent
