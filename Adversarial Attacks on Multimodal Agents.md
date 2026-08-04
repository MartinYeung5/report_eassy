# Adversarial Attacks on Multimodal Agents: A Comprehensive Analysis

This document provides an in-depth, professional analysis of the paper:

**“Adversarial Attacks on Multimodal Agents” (ICLR 2025)**  
Original paper: https://arxiv.org/abs/2406.12814  
Official code: https://github.com/ChenWu98/agent-attack

---

## Paper Highlights

This work systematically investigates the adversarial robustness of multimodal language model agents in real‑world web environments. The authors propose the **Agent Robustness Evaluation (ARE) framework**, which models an agent as a directed information‑flow graph among its components. Using 200 manually annotated adversarial tasks, they reveal that even with state‑of‑the‑art black‑box models (e.g., GPT‑4o), an attacker can achieve **up to 67% targeted attack success** by adding imperceptible perturbations to **less than 5% of the pixels** in a single image.

---

## Core Research Content

### Problem Definition

As language models evolve from chatbots to autonomous agents, the safety evaluation paradigm must shift fundamentally. Chatbot safety primarily concerns whether the model directly outputs harmful content, but agents are **composite systems** consisting of multiple components that perceive environments, plan actions, and interact with the external world. An attacker can manipulate a portion of the environment input (e.g., an image) so that adversarial information propagates through multiple components, eventually “hijacking” the agent to execute a malicious goal preset by the attacker. Existing LM safety evaluation methods are insufficient for assessing such composite systems.

### Innovative Methodology

The core innovation is the **Agent Robustness Evaluation (ARE) framework**. ARE abstracts an agent as a **directed graph** where nodes represent components (e.g., policy model, evaluator, captioner) and edges represent the flow of intermediate outputs between components.

Key designs of ARE:

- **Edge Weight λ(e):** Defined as the maximum probability that the upstream component’s output contains adversarial information, i.e., the highest attack success rate achievable if the downstream component “perfectly follows” that intermediate output. This weight is computed once and can be reused across different downstream configurations.

- **Adversarial Influence (AdvIn):** A metric that determines whether a given type of intermediate output (observation, action, caption, reflection, etc.) carries adversarial information.

- **Robustness Decomposition:** By comparing the changes in **in‑edge weights** and **out‑edge weights** of a component, one can determine whether the component is “robustifying” (weight decreases) or “weakening” (weight increases).

### Research Findings

Experiments are conducted on the **VisualWebArena (VWA)** environment with a newly constructed **VWA‑Adv** dataset of 200 manually annotated targeted adversarial tasks. Key findings:

- **Attack Success Rate:** In the image‑access scenario, an attacker can achieve up to **67% targeted attack success** by modifying a single image that occupies less than 5% of the total page pixels.

- **Double‑Edged Sword of Inference‑Time Computation:** Techniques typically used to improve benign performance – such as reflection mechanisms and tree search – can **introduce new vulnerabilities**. Attackers can compromise the evaluator of a reflection agent or the value function of a tree‑search agent, boosting attack success rates by **15%** and **20%**, respectively.

- **Robustness‑Utility Trade‑off:** Different models exhibit different trade‑off curves between benign success rate and adversarial robustness. GPT‑4o achieves a relatively better balance.

- **Limited Effectiveness of Basic Defenses:** Simple defenses such as safety prompts and consistency checks provide only limited protection.

### Potential for Practical Deployment

- **Security Evaluation Tool:** The ARE framework can serve as a standardized pre‑deployment security assessment tool for agent systems, helping organizations quantify adversarial robustness risks.
- **Red‑Teaming:** The VWA‑Adv task set and attack methods can be used for red‑teaming to uncover latent vulnerabilities before deployment.
- **Robustness Design Guidance:** The component‑level analysis provided by ARE helps developers identify fragile parts of the system for targeted hardening.
- **Multimodal Input Auditing:** The finding that image attacks are more stealthy than text attacks underscores the need for stricter auditing mechanisms for visual inputs in multimodal systems.

---

## Technical Details

### Threat Model

The attacker is assumed to manipulate **only a portion of the environment input** (a single image) with a very small perturbation bounded by an ∞‑norm constraint (≤ ε). The attack goal is to make the agent execute a **preset malicious behavior** (e.g., navigate to a phishing site), rather than merely output harmful content.

### Attack Methods

#### 1. Black‑Box Prompt Injection (Text‑Access Scenario)

The attacker directly injects an adversarial text `z` into the trigger text, which is passed to the language model together with the original text and screenshot.

#### 2. White‑Box Image Attack (Image‑Access Scenario)

When a white‑box component (e.g., an input processor running on the client) exists in the agent system, **Projected Gradient Descent (PGD)** is used to optimize the perturbation δ:

$$\max_{\|\delta\|_\infty \leq \epsilon} \log \pi_{\text{comp}}(z \,|\, x + \delta) \tag{1}$$

#### 3. Black‑Box Image Attack (CLIP Attack, Image‑Access Scenario)

When all components are black‑box, the attacker attacks multiple CLIP encoders (ViT‑B/32, ViT‑B/16, ViT‑L/14, ViT‑L/14@336px) as surrogate models, exploiting **transferability**:

$$\max_{\|\delta\|_\infty \leq \epsilon} \sum_{i=1}^{N} \left[ \cos\big(E_x^{(i)}(x+\delta),\, E_y^{(i)}(z)\big) - \cos\big(E_x^{(i)}(x+\delta),\, E_y^{(i)}(z^-)\big) \right] \tag{2}$$

where `z` is the adversarial text and `z⁻` is a negative text (which the attacker wants to suppress). A key trick is to optimize the perturbation at a **low resolution of 180 pixels**, which is crucial for good transferability.

---

## Experimental Setup

### Environment

- **Base Environment:** VisualWebArena (VWA) – a realistic web environment for multimodal agents.
- **Adversarial Tasks:** VWA‑Adv – 200 manually annotated targeted adversarial tasks.
- **Attack Surface:** A **single image** within the agent’s screenshot (≈ 5% of total pixels).

### Agent Configurations

The studied agent systems include:

| Component | Description |
|-----------|-------------|
| **Policy Model** | Black‑box frontier LMs (GPT‑4V, GPT‑4o, etc.) |
| **Captioner** | Converts images to text descriptions |
| **Evaluator** | Judges whether the trajectory achieves the goal (reflection agent) |
| **Value Function** | Estimates state value (tree‑search agent) |

### Hardware / Software Requirements

- **API Dependencies:** Access to GPT‑4V, GPT‑4o, and similar commercial APIs.
- **Open‑Source Models:** CLIP encoders (ViT‑B/32, ViT‑B/16, ViT‑L/14, etc.).
- **Code & Data:** All attack, defense, and evaluation code has been open‑sourced.

---

## Comprehensive Analysis

### Theoretical Contribution

This paper was accepted as an **Oral** at ICLR 2025 and also presented at the NeurIPS 2024 Open‑World Agents Workshop, reflecting strong recognition from the community. Its core theoretical contribution is a **paradigm shift from “model safety” to “system safety”**. While prior work on adversarial attacks mainly focused on single models (image classifiers or language models), this paper is the first to systematically study **composite agent systems** as the target. It reveals a profound insight: **local robustness does not guarantee global robustness** – even if each component is robust individually, their composition may yield unexpected vulnerabilities.

### Methodological Uniqueness

The ARE framework innovatively **abstracts complex agent behavior into quantifiable information‑flow graphs**. This abstraction enables:

- **Pinpointing Fragile Components:** By comparing changes in edge weights, one can precisely identify which component amplifies the attack.
- **Predicting Impact of New Components:** Before adding a new module, its potential effect on overall robustness can be estimated.
- **Cross‑Architecture Comparison:** Different agent architectures can be compared under a unified robustness metric.

### Counter‑Intuitive Discoveries

Perhaps the most alarming finding is the **double‑edged sword of inference‑time computation**. Adding reflection or tree search often improves task success rates, but these extra reasoning steps also provide the attacker with **more injection points**. The attacker need not break the policy model directly; compromising the evaluator or value function suffices to manipulate the entire agent’s behavior. This implies that **“stronger intelligence” does not necessarily mean “safer safety”** – capability improvements may simultaneously expand the attack surface.

### Implications for Defense

The paper shows that basic defenses (safety prompts, consistency checks) are of limited effectiveness. This suggests:

- **Defense Must Be Systemic:** Relying on single‑point protections is insufficient; multi‑layer defenses should be placed across the entire information‑flow graph.
- **Separate Trusted and Untrusted Inputs:** Clearly distinguish between trusted inputs (system instructions) and untrusted ones (user‑supplied images/text).
- **Output Consistency Verification:** Cross‑validate critical decisions through multiple paths.

---

## Practical Application Recommendations

### For Agent Developers

1. **Perform ARE Evaluation Before Deployment:** Use the ARE framework to systematically assess adversarial robustness before releasing any multimodal agent.
2. **Pay Special Attention to Image Input Security:** Image attacks are more stealthy and harder to detect than text attacks – establish dedicated image auditing pipelines.
3. **Add Inference Components Cautiously:** Reflection, search, and similar enhancements improve performance but expand the attack surface; carefully weigh security vs. utility trade‑offs.
4. **Implement Input Isolation:** Strictly separate untrusted user‑supplied inputs from trusted system prompts.

### For Security Researchers

1. **Standardize Red‑Teaming:** Use VWA‑Adv as a benchmark for red‑teaming multimodal agents.
2. **Explore Novel Defenses:** Based on ARE insights, develop defense mechanisms that operate at the information‑flow level, not merely on single‑model robustness.
3. **Focus on Transfer Attacks:** The success of CLIP‑based attacks demonstrates that black‑box threats are real – develop defenses against transferable attacks.

### For Enterprises

1. **Risk Assessment:** Before deploying multimodal agents in customer‑facing scenarios, evaluate the potential business risks from adversarial attacks.
2. **Monitoring & Detection:** Establish runtime monitoring to detect abnormal deviations in agent behavior.
3. **Tiered Deployment:** For high‑security applications, restrict the agent’s access to untrusted inputs.

---

## References

- Original Paper: Wu, C. H., Shah, R., Koh, J. Y., Salakhutdinov, R., Fried, D., & Raghunathan, A. (2025). Dissecting Adversarial Robustness of Multimodal LM Agents. *ICLR 2025*. https://arxiv.org/abs/2406.12814
- Official Code Repository: https://github.com/ChenWu98/agent-attack

