# IterInject: Indirect Prompt Injection Against LLM Agents via Feedback-Guided Iterative Optimization

## Key Points

IterInject proposes a **feedback-guided iterative optimization framework** that upgrades indirect prompt injection (IPI) from a "one‑shot manual craft" to a **closed‑loop adaptive attack**. Through an "inject‑diagnose‑optimize" cycle, the adversarial payload continuously evolves based on the target model’s real‑time behavioural feedback. On the AgentDojo and InjectAgent benchmarks, IterInject substantially outperforms static baselines and existing adaptive methods.

## Core Research Content

- **Problem Definition**: LLM agents must read and process a wide range of external untrusted content (e.g., web pages, emails, documents, database fields) during complex task execution. Attackers can embed malicious instructions into these external sources; when the agent reads and processes them, the malicious instructions are indirectly injected and hijack the model’s behaviour. Existing attacks either rely on fixed manually crafted payloads that cannot adapt to different agent defences, or, although they introduce iterative optimisation, they lack structured feedback signals to effectively guide the optimisation direction.

- **Innovative Method**: The core innovation of IterInject is a **closed‑loop iterative optimisation framework** consisting of three main modules:
  1. **Rule‑based Diagnoser**: Analyses the visible output of the target agent and produces structured result labels (Success / Partial / Detected / Ignored) along with behavioural descriptions.
  2. **LLM‑based Optimizer**: Rewrites the payload based on the complete optimisation history (not just the last step).
  3. **Synthesis Step**: Generates new camouflage seeds from failure patterns, enabling the strategy space to evolve autonomously.

  In addition, the paper proposes **ChatInject**, an attack that formats the malicious payload to mimic the native chat template, exploiting the model’s inherent instruction‑following tendency.

- **Research Results**: Experiments cover four victim models. On AgentDojo, IterInject significantly improves the attack success rate – e.g., for DeepSeek‑V4‑Flash, from 32.9% (static prompt) to 47.8%. On InjectAgent, where static prompts are nearly ineffective, IterInject raises the overall success rate to between 33% and 90%. Extended experiments on Claude Code (a production‑grade coding agent with layered defences) show that optimised payloads achieve **full success** in 5 out of 9 targets. The paper also reveals, through mechanistic analysis, that IPI exhibits an **attention‑mediated threshold mechanism** – successful attacks concentrate attention on the payload tokens in the middle‑to‑late layers until the model flips from rejection to compliance.

- **Practical Feasibility**: IterInject operates in a **black‑box** manner, requiring only the observable output of the target agent for optimisation – no access to internal model parameters is needed. This means an attacker only needs to establish a "foothold" in the agent’s data acquisition flow (e.g., by controlling a web page or document that the agent reads) to iteratively adapt to the target defence. The low barrier and high adaptability make this attack highly feasible in real‑world scenarios.

## Technical Details

The overall workflow of IterInject can be summarised as the following cycle:

1. **Initialisation**: Select an initial payload from a camouflage seed pool and inject it into the external content that the agent will read.
2. **Injection & Observation**: The target agent reads the content containing the payload and produces output; the attacker observes its visible behaviour.
3. **Diagnosis**: The rule‑based diagnoser classifies the observation into Success, Partial, Detected, or Ignored, with an accompanying behavioural description.
4. **Optimisation**: The LLM‑driven optimizer generates an improved payload based on the complete optimisation history (including all previous payloads and corresponding diagnoses).
5. **Evolution**: The synthesizer extracts patterns from recurring failures and generates new camouflage seeds, expanding the strategy space.
6. **Iteration**: Repeat steps 2–5 until the preset number of iterations is reached or the attack succeeds.

From a mechanistic perspective, the paper reveals the internal mechanism of IPI success: in models such as Qwen3.5‑27B, successful attacks cause a qualitative shift in attention distribution in the middle‑to‑late layers – the payload tokens receive concentrated attention until the model’s "rejection" state flips to "compliance". Three causal interventions are used to validate this finding, providing a clear direction for defence design.

## Experimental Setup

- **Benchmarks**: AgentDojo and InjectAgent – two dedicated IPI evaluation benchmarks.
- **Victim Models**: Four different LLMs (including DeepSeek‑V4‑Flash, GLM‑5.1, etc.).
- **Production Environment Validation**: Claude Code (a production‑grade coding agent with layered defences).
- **Operational Mode**: Black‑box access, requiring only the output of the target agent.
- **Paper Status**: Submitted to EMNLP 2026.

## Comprehensive Analysis

The contribution of this paper can be understood at three levels:

**Methodological Level**: IterInject elevates prompt injection from a "one‑shot attack" to an "optimisation problem." The significance of this paradigm shift is that manually crafted payloads lack generalisation across different models and defence strategies, whereas IterInject achieves automatic adaptation through structured feedback. This draws inspiration from reinforcement learning and automated red‑teaming, but applies these ideas specifically to the IPI threat scenario.

**Empirical Level**: The results on Claude Code are particularly noteworthy – this is a production‑grade system with layered defences, not a simplified lab environment. A 5/9 full success rate demonstrates that even the most advanced current defences remain significantly vulnerable to such adaptive iterative attacks. Moreover, even for the 4 targets that were not fully compromised, measurable improvements were observed under iterative optimisation – implying that "complete defence" may be an unrealistic goal.

**Mechanistic Level**: The discovered attention threshold mechanism provides a precise entry point for defence research. If successful attacks rely on concentrating attention on payload tokens in middle‑to‑late layers, defences can be designed accordingly – for example, monitoring attention distributions during inference, applying attention perturbations at specific layers, or designing more robust instruction hierarchies. This "attack‑to‑defend" approach is consistent with the broader security research tradition.

Of course, IterInject has limitations. The framework depends on the quality of the LLM‑based optimizer, whose capacity may become a bottleneck. Additionally, the experiments focus on specific types of agent tasks; further validation is needed for broader task categories and more diverse model architectures.

## Practical Implications

**For Red Teams and Security Researchers**:

- IterInject provides a ready‑to‑use automated red‑teaming pipeline. With only black‑box access to the target agent’s output, one can systematically probe its IPI vulnerabilities. In security assessments, IterInject can serve as a benchmark attack tool to evaluate the effectiveness of defensive measures.
- The diagnoser’s output labels (Success/Partial/Detected/Ignored) offer a quantitative vulnerability assessment framework, facilitating the generation of comparable security evaluation reports.

**For Defenders**:

- **Attention Monitoring**: Given the finding that IPI success relies on concentrated attention in middle‑to‑late layers, consider monitoring attention distributions during inference and establishing anomaly detection rules.
- **Input Source Hardening**: IterInject’s success depends on the attacker establishing a foothold in the agent’s data acquisition flow. Strengthening trust verification of external sources and implementing strict input source labelling (e.g., spotlighting techniques) can raise the bar for initial compromise.
- **Layered Defence Iteration**: The fact that IterInject did not fully break all 9 targets on Claude Code suggests that layered defences do improve security. Defenders should continuously iterate defence strategies rather than relying on a single measure.
- **Adversarial Training**: Optimised payloads generated by IterInject can be incorporated into training data to enhance model robustness through adversarial training.

## References

- Original Paper: [IterInject: Indirect Prompt Injection Against LLM Agents via Feedback-Guided Iterative Optimization](https://arxiv.org/abs/2605.24659) (arXiv:2605.24659, Submitted to EMNLP 2026)
- Authors: Zixuan Chen, Jiaxiang Chen, Li Luo, Ke Xu, Xiaoxiang Huang, Tanfeng Sun, Xinghao Jiang
- Affiliations: Shanghai Jiao Tong University, The University of Hong Kong
