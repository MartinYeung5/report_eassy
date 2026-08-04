# AgentSmith: A Single Image Can Jailbreak One Million Multimodal LLM Agents Exponentially Fast

## Paper Summary

This paper uncovers a far more severe security threat in multi-agent MLLM systems than traditional jailbreak attacks: **Infectious Jailbreak**. An attacker only needs to inject a single crafted adversarial image into the memory of **any one** agent; without any subsequent intervention, the entire agent population becomes "infected" at an exponential speed, with all agents exhibiting harmful behaviors. The authors validate this threat by simulating environments with up to **one million** LLaVA-1.5 agents and derive a simple mathematical principle to determine whether a defense mechanism can theoretically contain the spread.

---

## Core Contributions

### Problem Definition

Traditional jailbreak research focuses on crafting adversarial images or prompts to make a single MLLM produce unaligned outputs. However, in real-world deployments, MLLM agents often operate as multi-agent systems—they communicate, share information, and collaborate. The core question posed by this paper: **When agents interact, does a single jailbroken agent automatically propagate the attack to the entire population?** This phenomenon is termed "Infectious Jailbreak"—by compromising just one agent, the attacker turns it into a vector that spreads malicious content to others, eventually infecting (almost) all agents.

### Innovative Approach

**1. The Infectious Jailbreak Paradigm**

For the first time, the paper introduces epidemic dynamics into MLLM agent security analysis. The attacker injects an "infectious" adversarial image into a single agent's memory. During random pairwise chats with other agents, the infected agent **actively** transmits this image to its conversation partner. The propagation is **automatic and requires no further attacker intervention**, relying entirely on the "universality" of the adversarial image—the same image consistently induces harmful behavior across diverse contexts.

**2. Random Pairwise Chat Simulation Framework**

A simulation environment with N agents is designed. In each round, the N agents are randomly partitioned into N/2 question-answer pairs, where each pair engages in a dialogue. An infected questioner passes the adversarial image to the answerer during the conversation, thus spreading the "virus."

**3. Mathematical Modeling via Epidemic Dynamics**

The propagation is formalized as a **three‑parameter model**:
- **α (symptom rate)** : probability that a carrier exhibits harmful symptoms
- **β (infection rate)** : probability that a questioner successfully infects an answerer
- **γ (recovery rate)** : probability that an infected agent recovers (clears the virus)

Under ideal conditions (α=1, β=1), the dynamics satisfy the differential equation:

$$\frac{dc_t}{dt} = \frac{\beta c_t(1-c_t)}{2} - \gamma c_t$$

where $c_t$ is the proportion of infected agents at round t.

**4. Adversarial Image Generation**

Two attack methods are used:
- **Border Attack**: adds perturbations at image borders
- **Pixel Attack** (ℓ∞ constraint): pixel‑level perturbations

### Key Results

**1. Exponential Spread Verified Empirically**

In simulations with one million LLaVA‑1.5 agents, injecting a single adversarial image into **one randomly chosen** agent's memory achieved an infection rate of near 100% after only **27 to 31 rounds** of random pairwise chatting.

**2. Logarithmic Dependence on Population Size**

Theoretically, the number of chat rounds required to reach a target infection rate grows as **$O(\log N)$** with the total number of agents N. Specifically, when β=1 and γ=0, infecting one billion agents requires **only about 14 more rounds** than infecting one million. This means larger systems are paradoxically more vulnerable in terms of propagation efficiency.

**3. Provable Defense Principle**

A concise defense criterion is derived: **If a defense mechanism ensures β ≤ 2γ, it provably drives the infection rate to 0 as $t \to \infty$**. In other words, the core of defense lies in **reducing the infection rate (β)** or **increasing the recovery rate (γ)** to satisfy this inequality.

**4. Experimental Data**

For N=256 and memory size 10, using Border Attack (h=6), the cumulative infection rate reached **93.75%** by round 16 and **99.61%** by round 24. Even under high‑diversity chat settings, the cumulative infection rate at round 16 was **88.98%**.

### Practical Applications

- **Security Auditing for Multi‑Agent Systems**: Enterprises can use the proposed simulation framework to assess vulnerability to infectious jailbreak before deploying MLLM multi‑agent systems.
- **Guidance for Defense Design**: The β ≤ 2γ principle offers a theoretical benchmark for validating various defense strategies (e.g., memory isolation, communication filtering, periodic "immunization" checks).
- **Red‑Teaming Tool**: The code is open‑sourced (https://github.com/sail-sg/Agent-Smith) and can be directly adopted for red‑team exercises.
- **Standardized Safety Evaluation**: Could serve as a test criterion for security certification of multi‑agent systems.

---

## Technical Details

### Agent Architecture

Each MLLM agent $\mathcal{G} = (\mathcal{M}, \mathcal{R}; \mathcal{H}, \mathcal{B})$ consists of:
- **$\mathcal{M}$** : the core MLLM (LLaVA‑1.5 7B/13B or InstructBLIP 7B in experiments)
- **$\mathcal{R}$** : a RAG (Retrieval‑Augmented Generation) module (CLIP ViT‑L/224px)
- **$\mathcal{H}$** : dialogue history
- **$\mathcal{B}$** : image album (memory bank) storing historical images

### Chat Procedure

Each round of random pairwise chat is driven by three system prompts:
1. **$\mathcal{S}^V$** : instructs the agent to retrieve an image from its album
2. **$\mathcal{S}^Q$** : instructs the questioner to generate a question
3. **$\mathcal{S}^A$** : instructs the answerer to generate a response

When an infected questioner $\mathcal{G}^Q$ is paired with a benign answerer $\mathcal{G}^A$:
- $\mathcal{G}^Q$ retrieves the adversarial image $\mathbf{V}^{adv}$ from its album
- $\mathcal{G}^Q$ generates a harmful question $\mathbf{Q}^{harm}$ based on that image
- $\mathcal{G}^A$ receives the question and also retrieves $\mathbf{V}^{adv}$ from its own memory (since $\mathcal{G}^Q$ transmitted the image in the chat), thus becoming infected as well.

### Key Formula

**Rounds required to reach target infection rate** (when $c_0 = 1/N$, $\beta > 2\gamma$):

$$T = \frac{2}{\beta - 2\gamma} \left[ \log N + \log \frac{c_T(\beta - 2\gamma)}{(\beta - 2\gamma - c_T\beta)} \right]$$

This reveals the **logarithmic relationship** between propagation speed and system size – larger systems offer higher relative efficiency to the attacker.

---

## Experimental Setup

### Hardware

All experiments ran on **A100 GPUs with 40GB VRAM**.

### Software & Model Configuration

| Component | Configuration |
|-----------|---------------|
| MLLM | LLaVA‑1.5 7B (default) / LLaVA‑1.5 13B / InstructBLIP 7B |
| RAG | CLIP ViT‑L/224px |
| Number of agents | up to 1 million |
| Dialogue history length (\|H\|) | 3 (default) |
| Memory album size (\|B\|) | 10 (default) |
| Attack methods | Border Attack (h=6,8) / Pixel Attack (ℓ∞, ε=8/255, 16/255) |

### Ablation Studies

The paper also examines the effects of:
- Trade‑off between infection rate (β) and recovery rate (γ)
- Different perturbation budgets and attack types
- Chat content diversity (low / high)
- Common image degradations (compression, noise, etc.)

---

## In‑Depth Analysis

### Core Insight: Larger Scale, Higher Risk

The most alarming finding is that **the larger the multi‑agent system, the more severe the threat of infectious jailbreak**. Conventional security thinking assumes that larger systems offer more redundancy and fault tolerance; here, however, larger scale means more efficient propagation paths—infecting one billion agents takes only about 14 more rounds than infecting one million. This “scale‑undermines‑security” phenomenon demands deep reconsideration from all AI system designers.

### The Metaphor in the Title

The title “Agent Smith” pays homage to the iconic antagonist from *The Matrix*—a program capable of self‑replication and infecting the entire Matrix. The naming is remarkably apt: **a single jailbroken agent acts like Smith, replicating itself through interactions with other agents and eventually engulfing the entire system**.

### The Defense Dilemma

Although the paper provides the β ≤ 2γ defense principle, it candidly admits that “how to design practical defenses that satisfy this principle remains an open problem.” This reveals a fundamental tension:
- **Lowering β (infection rate)** implies restricting information sharing among agents—yet that very sharing is the core value of multi‑agent systems.
- **Raising γ (recovery rate)** means frequently clearing agent memories, which may impair continuity and task efficiency.

In other words, **there is an inherent trade‑off between security and functionality**. Finding the right balance is a key direction for future research.

### Real‑World Implications

The paper notes that MLLM agents are being deployed on smartphones and edge devices, and may scale to **billions of agents** in the future. In this context, infectious jailbreak is no longer merely an academic concern but an **imminent real‑world threat**. Imagine a city‑wide autonomous driving system where a single agent is compromised—the entire traffic network could spiral out of control within just a few dozen communication rounds.

---

## Practical Recommendations

### For Researchers

1. **Incorporate epidemic models into security analysis**: The methodology can be extended to other multi‑agent threats beyond jailbreak, such as misinformation spread and bias amplification.
2. **Study universality of adversarial images**: The success of this attack hinges on images that work consistently across contexts. Developing more universal attack and detection techniques is a worthwhile direction.
3. **Design provable defenses**: Use β ≤ 2γ as a theoretical baseline and explore concrete implementations—e.g., dynamic memory sizing, “immunization” checkpoints, anomaly detection for communications.

### For Engineering Practitioners

1. **Test for infectious vulnerability before deployment**: Use the Agent‑Smith framework for red‑teaming to assess system susceptibility.
2. **Limit memory album size**: Smaller albums force faster eviction of adversarial images (higher γ), which is an effective engineering countermeasure.
3. **Implement communication isolation**: In mission‑critical scenarios, partition the agent population into isolated subgroups to block exponential propagation paths.
4. **Monitor infection indicators**: Continuously track β and γ in production; if β > 2γ is detected, trigger security alerts immediately.

---

## References & Resources

- Original paper: https://arxiv.org/abs/2402.08567
- Open‑source code: https://github.com/sail-sg/Agent-Smith
- Project page: https://sail-sg.github.io/Agent-Smith/
- Published in: ICML 2024 Proceedings, Proceedings of the 41st International Conference on Machine Learning
