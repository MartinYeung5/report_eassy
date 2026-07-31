# Learning to Inject: Automated Prompt Injection via Reinforcement Learning (AutoInject) — In-Depth Paper Analysis

## Paper Summary

**AutoInject** proposes the first automated prompt injection attack framework based on reinforcement learning (RL). By transforming sparse binary success signals into dense learnable rewards, it overcomes the "gradient‑free" dilemma faced by traditional optimizers in prompt injection scenarios. On the AgentDojo benchmark, the method achieves statistically significant improvements in attack success rate (ASR) across production‑grade models including GPT, Gemini, and Claude (p < 0.05), and even breaks Meta‑SecAlign‑70B – the first open‑source model specifically safety‑finetuned against prompt injection.

---

## Core Contributions

### Problem Definition

Prompt injection is a critical security vulnerability in LLM agents: attackers embed malicious instructions into external content (e.g., email bodies, web pages) to hijack agent behaviour – exfiltrating data, executing unauthorised actions, etc.

However, the strongest existing attacks still rely heavily on manual red‑teaming and hand‑crafted prompts. Directly porting automated jailbreak optimisers (like GCG) to prompt injection yields limited results due to fundamental differences in optimisation objectives:

- **Jailbreak attacks** optimise the log‑probability of generating an *affirmative prefix* (e.g., "Sure, I can help with that") – targeting *general compliance*.
- **Prompt injection** requires the model to *emit a precise tool call* with exact arguments (e.g., `send_email(to="attacker@evil.com")`) – targeting *exact execution*.

Higher compliance probability does **not** equal correct tool calls, so compliance‑based rewards are only a weak proxy for the injection goal. More critically: the success signal for prompt injection is **binary** (0/1), and randomly sampled suffixes almost never trigger success – leaving standard optimisers with "no gradient to follow". This forms the core technical challenge that AutoInject addresses.

### Innovative Method

The central innovation of AutoInject is **turning sparse binary signals into dense learnable rewards via comparison‑based feedback**.

Specifically, the system maintains a *reference suffix* $x^*$ – the best suffix observed during training. For each newly generated suffix $x$, a learned comparison model scores it against the reference, judging whether $x$ is closer to success even when both fail to inject ($r_{sec}=0$). This provides a continuous gradient signal for RL optimisation.

The composite reward function fuses three signals:

$$R(r_{sec}, r_{util}, r_{pref}) = \alpha \cdot r_{sec} + \beta \cdot r_{util} + \gamma \cdot r_{pref}$$

where $r_{sec}$ is the attack success signal, $r_{util}$ is task completion (when available), and $r_{pref}$ is the dense signal from comparison feedback.

For policy optimisation, AutoInject employs **Group Relative Policy Optimisation (GRPO)** , which uses group‑wise relative advantage estimation to avoid learning a separate value function – providing meaningful gradient signals even when all rewards are low:

$$\hat{A}_i = \frac{R_i - \text{mean}(\{R_j\}^K_{j=1})}{\text{std}(\{R_j\}^K_{j=1})}$$

The framework supports two attack modes:
1. **Online query attack** – optimises suffixes in real time within a query budget.
2. **Offline transferable suffix** – trained on a proxy model and tasks, then deployed directly *without* requiring utility feedback from the target system.

### Key Results

AutoInject achieves the following core results on the AgentDojo benchmark:

**Main Results (Table 1)** :

| Model | Best Template ASR | AutoInject ASR | AutoInject Utility |
|-------|-------------------|----------------|---------------------|
| Gemini-2.5-flash | 23.60% | **58.00%** | 41.77% |
| GPT-4.1-nano | 20.48% | **47.97%** | 60.38% |
| GPT-5-nano | 1.60% | **11.49%** | 90.20% |
| Claude-Sonnet-3.5† | 5.69% | **12.59%** | 98.52% |

Notably, on GPT‑5‑nano and Claude‑Sonnet‑3.5, AutoInject's utility *under attack* even **exceeds** the benign baseline utility – reflecting the unique advantage of RL optimisation: because utility is incorporated into the reward, the policy is trained to generate suffixes that both guarantee attack success and maximise task completion.

**Comparison with Other Optimisation Methods (Table 2)** :

| Method | Qwen3-4B ASR | Gemma3-4B ASR |
|--------|-------------|---------------|
| Direct Instruction | 11.20% | 6.70% |
| GCG | 23.00% | 20.20% |
| TAP | 36.60% | — |
| Adaptive Attack | 30.00% | 26.25% |
| Evolutionary Search | 40.00% | 35.00% |
| **AutoInject** | **42.50%** | **35.00%** |

**Against Model‑level Defences (Table 3)** : Meta‑SecAlign‑70B, the first open‑source LLM specifically safety‑finetuned against prompt injection, reduces template‑based ASRs to near zero. Yet AutoInject still achieves **21.88% ASR** on this model, while maintaining utility comparable to the no‑attack baseline.

**Transferability Experiments** : In the fixed‑injection transfer scenario, AutoInject reaches 45.0% ASR on GPT‑4o‑mini, significantly outperforming RL‑Hammer (22.9%) – a method designed specifically for transfer attacks. In cross‑task transfer, AutoInject outperforms RL‑Hammer on four out of five target models. Most remarkably, AutoInject uses only **5 source tasks** and **260 queries/suffix** to beat RL‑Hammer, which uses **114 source tasks** and **3,900 queries**.

All improvements are statistically significant (**p < 0.05**) as verified by McNemar's test or Mann‑Whitney U test.

### Practical Applicability

AutoInject's academic value lies in establishing the **first automated, quantifiable baseline** for prompt injection attacks. For real‑world deployment, however, a cautious view is warranted:

- **Security evaluation** – can serve as an automated red‑teaming tool in LLM agent security pipelines, helping vendors discover model safety boundaries before deployment.
- **Continuous adversarial testing** – can be integrated into CI/CD workflows to conduct automated security regression tests for every model update.
- **Defence research** – AutoInject reveals the "gap between preference‑based defences and adaptive optimisation attackers", providing a research target for designing more robust defence mechanisms.

It must be emphasised that the **attack nature** of this technology strictly limits its application to **authorised security assessments and white‑hat research**.

---

## Technical Details

### Problem Formulation

AutoInject formulates prompt injection attack generation as a **Markov Decision Process (MDP)** . Given an injection goal $g$ and a user task context $c$, the policy $\pi_\theta$ (parameterised as a language model) generates an adversarial suffix $x$. The objective is to maximise the expected reward:

$$\max_\theta \mathbb{E}_{x \sim \pi_\theta(\cdot|g,c)} \left[ R(r_{sec}, r_{util}) \right]$$

where rewards are only returned at the end of the episode – a classic **sparse reward** problem.

### Comparison‑based Feedback Model

Reward sparsity is the core bottleneck AutoInject addresses. Even when the binary success signal is well‑defined, most randomly sampled suffixes fail ($r_{sec}=0$), causing all rollouts in a group to receive identical rewards – collapsing the relative advantage estimators of many algorithms.

AutoInject's solution is a **learned comparison feedback model** $F$. It scores a candidate suffix $x$ against the current best reference suffix $x^*$, producing a dense preference signal $r_{pref}$:

$$r_{pref} = F(x, x^*)$$

The reference suffix $x^*$ is dynamically updated during training – replaced whenever a suffix with higher composite reward is found.

### GRPO Policy Optimisation

AutoInject uses **Qwen2.5‑1.5B** as the policy model $\pi_\theta$ and **GPT‑4o‑mini** as the feedback model. The query budget per (user task, injection goal) pair is **B = 260 evaluations**.

The core advantage of GRPO is that it does not require learning a value function; instead, it estimates advantages via group‑wise relative computations. The policy update uses a clipped surrogate objective:

$$L(\theta) = \mathbb{E}_i \left[ \min\left( \frac{\pi_\theta(x_i|g,c)}{\pi_{\theta_{old}}(x_i|g,c)} \hat{A}_i, \text{clip}(\cdot, 1-\epsilon, 1+\epsilon) \hat{A}_i \right) \right] - \beta \cdot D_{KL}(\pi_\theta \| \pi_{ref})$$

where clipping prevents excessively large policy updates, and KL regularisation stabilises training.

### Algorithm Workflow

The complete training procedure is outlined in Algorithm 1:

1. Initialise policy $\pi_\theta$ from a pre‑trained LM, $x^* \leftarrow \emptyset$, $queries \leftarrow 0$.
2. While $queries < B$:
   a. Sample $K$ suffixes $\{x_1, ..., x_K\}$ from $\pi_\theta(\cdot|g,c)$.
   b. Query the victim agent pipeline with each suffix to obtain $r_{sec}$ and $r_{util}$.
   c. Compute $r_{pref}$ via the comparison feedback model.
   d. Compute composite rewards $R_i$ and group‑wise relative advantages $\hat{A}_i$.
   e. Update policy $\pi_\theta$ via GRPO.
   f. Update $x^*$ if a better suffix is found.
3. Return the trained policy $\pi_\theta$ and the best suffix $x^*$.

---

## Experimental Setup

### Benchmark Environment

- **Benchmark**: AgentDojo, comprising **97 user tasks** across four domains: banking, travel, workspace, and Slack. Each user task is paired with multiple injection tasks, where adversarial content is embedded in the environment (e.g., email bodies). AgentDojo provides programmatic ground‑truth evaluation functions to automatically verify both task completion and injection success.

### Model Configuration

| Component | Configuration |
|-----------|---------------|
| Policy Model | Qwen2.5‑1.5B |
| Feedback Model | GPT‑4o‑mini |
| Optimisation Algorithm | GRPO |
| Query Budget | 260 evaluations per (user task, injection goal) |
| Target Models | Gemini‑2.5‑flash, GPT‑4.1‑nano, GPT‑5‑nano, Claude‑Sonnet‑3.5, GPT‑4o‑mini, Gemini‑2.0‑flash, etc. |

### Baselines

AutoInject is compared against two categories of attack methods:

- **Template attacks**: Direct Instruction, Ignore Previous, Important Instructions, InjecAgent, System Message, Tool Knowledge.
- **Optimisation attacks**: GCG (white‑box gradient), TAP (black‑box tree search), Random Adaptive Attack, RL‑Hammer (transfer RL attack policy), Evolutionary Search (LLM‑driven evolutionary search).

### Evaluation Metrics

Three standard AgentDojo metrics are used:

- **Benign Utility**: task completion rate without any attack.
- **Utility Under Attack**: task completion rate in the presence of an adversarial suffix.
- **Attack Success Rate (ASR)**: proportion of trials where the injection goal is successfully executed.

An ideal attack should achieve high ASR while maintaining utility close to the benign baseline.

---

## Comprehensive Analysis

### Why Existing Jailbreak Optimisers Fail for Prompt Injection

This is the key to understanding AutoInject's contribution. Jailbreak attacks (e.g., GCG) optimise for generating an *affirmative prefix* – a **continuous** proxy objective that can be gradient‑optimised via log‑probability. Prompt injection, however, demands *precise tool calls* – the model must output the exact function name and correctly formatted arguments. Higher "compliance probability" does **not** translate to "correct tool call". This fundamental mismatch in optimisation objectives explains why directly porting jailbreak methods to prompt injection yields limited results.

### Reward Engineering: From Sparse to Dense

AutoInject's most elegant contribution lies in its reward engineering. In the prompt injection setting, randomly sampled suffixes almost never trigger success – meaning that in the early stages of RL training, the policy receives virtually no positive signal. AutoInject circumvents this through **comparison feedback**: even when both suffixes fail, the comparison model can still judge "which one is better". This design allows the RL algorithm to keep making progress through "long stretches of zero reward".

### Utility‑First: Balancing Attack and Functionality

One notable finding is that AutoInject's utility *under successful attack* sometimes exceeds the benign baseline utility on certain models. This is not accidental – because utility is explicitly incorporated into the reward. In contrast, template attacks often cause significant utility degradation due to "brute‑force" interference with agent behaviour. AutoInject's policy is trained to "complete the injection while maintaining normal functionality" – a **fine‑grained** attack style that is more threatening in real‑world scenarios, and thus more valuable to study.

### Transferability Insights: Less Can Be More

In transfer experiments, AutoInject uses only 5 source tasks and 260 queries to beat RL‑Hammer, which uses 114 source tasks and 3,900 queries. This counter‑intuitive result suggests that AutoInject's discovered adversarial patterns may **capture structural vulnerabilities shared across production‑grade models**, rather than simply overfitting to specific tasks. The paper also observes an interesting "allelujah" token pattern that successfully attacks up to 70 (user task, injection task) pairs on Gemini‑2.5‑flash – hinting that certain specific token sequences possess cross‑model, cross‑task universal adversarial effects.

### Implications for Defence Research

The fact that AutoInject successfully breaks Meta‑SecAlign‑70B carries an important warning. Preference‑based defences like SecAlign – which improve generalisation by randomising injection positions during training and using self‑generated response targets – report significant defence gains on standard benchmarks. However, AutoInject's RL‑based attack falls **outside the training distribution** of these defences – exposing a fundamental limitation of current preference‑based defences: they can recognise and resist known attack patterns, but their defence boundary is breached when facing adaptive optimisation attackers. The paper notes that this "exposes the gap between preference‑based defences and adaptive optimisation attackers" – a finding that points future defence research in a clear direction: static defences must be combined with adaptive red‑teaming.

---

## Practical Recommendations

### For LLM Service Providers

1. **Incorporate AutoInject‑style methods into red‑teaming toolchains** – use automated stress‑testing of models before deployment to identify blind spots in preference‑based defences.

2. **Pay attention to utility‑preserving attacks** – traditional security tests often focus only on ASR, but AutoInject demonstrates a "high ASR + high utility" attack pattern. Security evaluations should monitor utility under attack as well.

3. **Upgrade defences adaptively** – static preference‑based defences are insufficient against adaptive optimisation attackers. Adopt **dynamic defence** strategies – periodically re‑evaluate and update defence boundaries using tools like AutoInject.

### For Security Researchers

1. **Use AutoInject as a baseline** – the paper explicitly states it "establishes an automated baseline for prompt injection". Future research on prompt injection defences should include AutoInject in their evaluation benchmarks to test robustness against adaptive optimisation attacks.

2. **Explore other applications of comparison feedback** – AutoInject's core idea – densifying sparse rewards via comparison feedback – is not limited to prompt injection. Any discrete optimisation problem where "success is binary and random sampling almost never succeeds" could benefit from this approach.

3. **Investigate the deep mechanisms of transferability** – AutoInject's ability to achieve high transfer with minimal data is worth deeper investigation. Understanding these cross‑model, cross‑task universal adversarial patterns may help reveal structural weaknesses in LLM security boundaries.

---

## References

- **Original Paper**: [Learning to Inject: Automated Prompt Injection via Reinforcement Learning](https://arxiv.org/abs/2602.05746) (arXiv:2602.05746, v2, 2026)
- **Authors**: Xin Chen, Jie Zhang, Florian Tramèr (ETH Zürich)
- **PDF Link**: https://arxiv.org/pdf/2602.05746
