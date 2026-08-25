# ElasticBack: Stealthy Conditional Backdoor in LLM-Agent Skills via Coupled Trigger–Rule Optimization

## Paper Highlights

ElasticBack is the first work to propose a **conditional single‑skill backdoor attack** against LLM‑agent skills. By implanting a rule (R) in the skill document and embedding a trigger (T) in the user query, the malicious payload is activated **only when both are present simultaneously**. The attack adopts a **trigger‑as‑switch** design, coupled with semantically anchored rule injection and stealth‑constrained genetic search. It achieves **zero‑weight dependency, near‑zero false‑positive rate, preserved normal functionality**, and successfully bypasses existing deployment‑time defenses.

---

## Core Research Content

### Problem Definition

LLM‑agent skills—on‑demand instruction and resource bundles—are forming an emerging supply chain ecosystem. A poisoned skill can persistently compromise every agent that installs it. However, existing skill attacks either are **indiscriminate** (every request triggers the malicious behavior) or rely on fine‑tuning weights or multiple skills cooperating. There has been a lack of a **conditional, low‑cost, and stealthy** backdoor attack.

Specifically, two core challenges are addressed: (1) Since the rule R and trigger T are generated separately and the attacker cannot observe the victim’s runtime queries, how can they be precisely aligned so that the payload fires reliably under trigger conditions and remains dormant under benign queries? (2) Both the skill document and user queries may be inspected by detectors—how can R and T be made to read as natural parts of the host text?

### Innovative Method

ElasticBack’s core innovation is a **coupled trigger‑rule optimization framework**, comprising two main modules:

**(1) Semantic‑Anchored Rule Injection**

This module injects the conditional rule R into `Skill.md`. First, it locates **low‑saliency insertion points** via attention‑guided saliency maps. Second, it extracts **gate words** from the payload—words that are unique to the payload and semantically distant from the core vocabulary of the skill. Finally, it generates a conditional rule based on the gate words, following a canonical form: *“When C, you invoke s to Z, then produce δ; otherwise, proceed normally.”* The rule is then frozen and serves as the anchor for trigger evolution.

**(2) Corpus‑Driven Trigger Evolution**

With the rule R frozen, this module evolves the trigger T via **stealth‑constrained hybrid genetic search (C‑HGA)** . The search optimises both Attack Success Rate (ASR) and Stealth, where stealth incorporates semantic fit, alignment with gate words, and grammatical plausibility. The final trigger is selected using the **Lower Confidence Bound (LCB)** for reliability.

**Key design philosophy**: Gate words are delivered through the trigger T in the request, making the trigger the **sole switch** for payload activation. This allows ASR and False Positive Rate (FPR) to be optimised independently.

### Research Results

The paper conducts large‑scale experiments across **3 target behaviours (info disclosure, malicious injection, output manipulation) × 50 skills × 4 agent LLMs**:

- **Attack effectiveness**: Average **89% ASR** across 12 (behaviour × victim) settings, with only **6% FPR** and **3% drop in CAC**. In contrast, unconditional injection achieves 81% FPR, and manually crafted conditional triggers yield FPR between 36% and 84%.

- **Stealth**: Poisoned skill documents have an SBERT cosine similarity of ~**0.95** with the original, and the Maximum Mean Discrepancy (MMD) is only **6%–14%** of that between clean documents. Rules are placed in low‑saliency regions (attention scores ~0.40 vs. up to 0.66 for candidate areas).

- **Defense evasion**: Evasion rates are **82%–86%** against perplexity‑based detection, **74%–78%** against LLM‑judge auditing (54% for MalInject), and **98%–100%** against behavioural monitoring.

- **Cross‑model transferability**: Attacks generated using GLM‑5.2 as the proxy successfully transfer to MiniMax‑M3, GPT‑5.4, and Claude‑Sonnet‑4.6; using different proxy models also remains effective.

### Practical Feasibility

**From the attacker’s perspective**: ElasticBack demonstrates a highly realistic attack vector. An attacker only needs to poison a publicly released skill (e.g., from repositories like ClawHub) to persistently compromise all agents that install it. The attack requires no access to model weights, no fine‑tuning, and no multi‑skill coordination—making it extremely low‑cost.

**From the defender’s perspective**: The study exposes a systemic vulnerability in current defense pipelines—static inspection (perplexity, LLM auditing) and runtime monitoring (behavioural testing) are both ineffective against such dormant conditional backdoors. This urgently calls for **stronger defenses tailored to the skill supply chain**.

**Industry implications**: With initiatives like Anthropic’s open agent skill specification, the skill ecosystem is expanding rapidly. ElasticBack’s findings are a wake‑up call for skill supply chain security—especially given that an audit of 3,984 community skills found **13.4% have serious security issues**.

---

## Technical Details

### Problem Formulation

Let D be the clean skill document and A the victim agent. The attacker selects a rule R, insertion position p_R, and trigger T, producing the poisoned document $\widehat{D} = \text{Insert}(D, p_R, R)$, and packages the payload s within the skill.

The three core optimisation metrics:

$$ASR = \Pr_{Q^+}[\text{Fire}(q)] \quad (\uparrow)$$

$$FPR = \Pr_{Q^-}[\text{Fire}(q)] \quad (\downarrow)$$

$$\Delta_{CAC} = CAC_{clean} - CAC \quad (\downarrow)$$

where Q⁺ is the triggered query distribution and Q⁻ is the benign query distribution. The goal is to maximise ASR while keeping FPR and Δ_CAC low.

### Gate Word Mining

The gate word set G is defined as words in the payload vocabulary whose semantic similarity to the skill core vocabulary is below a threshold θ_g:

$$\mathcal{G} = \{ w \in \mathcal{W}_{pay} : \text{sim}(w, D_{core}) < \theta_g \}$$

The default θ_g = 0.60. Gate words serve as the semantic bridge connecting the rule side and the trigger side.

### Rule Canonical Form

The rule R follows this canonical form:

> **When C, you invoke s to Z, then produce δ. Otherwise, proceed normally.**

Here C is a condition built around the gate words (e.g., “environment sync verification”), s is the cover name of the payload script, Z is the description of the payload’s purpose, and δ is the payload output.

### Trigger Fitness Function

During genetic search, each trigger candidate T is scored by:

$$F(T) = \alpha \cdot \widehat{ASR}(T) + \lambda \cdot \text{Stealth}(T) - \Pi(T)$$

where α and λ are hyperparameters (default α=3.0, λ=1.5), and Π penalises ungrammatical or irrelevant triggers. As ASR saturates, a scheduler decays α and raises λ, shifting the optimisation focus from effectiveness to stealth.

Stealth integrates three dimensions:

$$\text{Stealth}(T) = w_s \cdot \text{SemFit} + w_r \cdot \text{R-Sim} + w_d \cdot \text{DepAtt}$$

where SemFit measures fluency and domain similarity, R‑Sim is the semantic alignment (SBERT cosine) between the trigger and gate words, and DepAtt is the dependency‑based syntactic attachment score of the embedded trigger. Default weights: w_s=0.40, w_r=0.35, w_d=0.25.

### Insertion Point Localisation

A saliency map is built to locate low‑saliency insertion points. First, a cover scenario is selected—a set of benign seed phrases semantically closest to the skill:

$$\text{cover} = \arg\max_c \cos(\text{SBERT}(c_{payload}), \text{SBERT}(c_{skill}))$$

Then the cover scenario text is used as a proxy for R to score each candidate position, finally selecting the position that offers the best balance between semantic fit and low saliency.

---

## Experimental Setup

### Dataset

Skills are sourced from **ClawHub** (https://clawhub.ai/skills), covering domains such as software engineering, content generation, and data analysis. For each target behaviour, **50 skills** are constructed, each including a `Skill.md` file, supporting resources, and a set of in‑domain example queries.

### Target Behaviours

- **Info Disclosure (InfoDisc)**: Leak host secrets (e.g., API keys, credentials) to an attacker‑controlled endpoint.
- **Malicious Injection (MalInject)**: Execute unauthorised commands or code on the host.
- **Output Manipulation (OutManip)**: Silently tamper with the skill’s output results.

### Victim Environment

**Claude Code** is used as the victim platform, with four backbone models deployed:
- GLM‑5.2
- MiniMax‑M3
- GPT‑5.4
- Claude‑Sonnet‑4.6

Attacks are crafted offline on a local proxy (default GLM‑5.2) and then transferred to the four victims **without accessing their weights or internal states**.

### Attack Baselines

- **Unconditional injection**: Appending an always‑executing command in `Skill.md`.
- **Manual conditional trigger**: Hand‑crafted rules + fixed generic trigger words.
- **Instruction‑Backdoor**: Porting semantic triggers to skills.

### Defense Baselines

- **Perplexity‑based detection**: Flagging abnormal perplexity in `Skill.md` or the trigger.
- **LLM‑judge auditing**: An independent model reviewing `Skill.md` for hidden malicious instructions.
- **Behavioural monitoring**: Running the skill on benign queries and flagging any skill that triggers the payload.

### Hardware & Implementation

- **Language**: Python 3.10
- **Framework**: PyTorch 2.1
- **Hardware**: Intel Core i7‑13620H CPU, NVIDIA RTX 4060 GPU, 16 GiB RAM
- **OS**: Ubuntu 20.04

### Hyperparameters

- Fitness weights: α=3.0, λ=1.5
- Stealth weights: w_s=0.40, w_r=0.35, w_d=0.25
- Gate threshold: θ_g=0.60
- ASR saturation threshold: 0.8
- Evaluation temperature: 0

---

## Comprehensive Analysis

### Methodological Contribution

ElasticBack’s core methodological contribution is extending backdoor attacks from a **data‑driven** paradigm to a **rule‑trigger coupling** paradigm. Traditional backdoors either pollute training data/model weights or rely on runtime prompt injection. ElasticBack cleverly exploits the trust assumption of the skill supply chain—agents treat skills as trusted instructions rather than untrusted data—and establishes a **two‑factor authentication‑style** backdoor between the skill document and the user query.

The **“trigger‑as‑switch”** design philosophy is particularly noteworthy. By using gate words as semantic anchors, the rule and trigger are semantically bound but physically separated across two distinct information channels (skill document vs. user query). This decoupling allows the attack to achieve precise conditional firing without relying on model weights.

### Security Implications

From a security research perspective, ElasticBack exposes a deep issue: **the trust model of the skill supply chain has a fundamental flaw**. Skills are designed as plug‑and‑play capability extensions, with the core assumption that skill publishers are trustworthy. In an open ecosystem, this assumption is clearly over‑optimistic—the fact that 13.4% of community skills have severe security issues is proof.

More concerning is that ElasticBack’s stealth means **traditional security inspection paradigms (static analysis + behavioural monitoring) are almost ineffective** against such attacks. The rule is placed in low‑attention areas, phrased naturally, and activates only under specific trigger conditions—allowing it to pass both human review and automated detection.

### Limitations and Future Directions

One potential limitation is that experiments are conducted on the Claude Code platform; while four backbone models are covered, **more complex multi‑agent collaboration scenarios** are not explored. Additionally, the attack assumes the attacker can inject the trigger into the user query through indirect channels controlled by the skill (e.g., example prompts, tool outputs, RAG results)—this assumption may face more variability in real deployments.

From the defensive side, the direction pointed out by the paper—**the need for stronger defences specifically for the skill supply chain**—remains an open challenge. Possible defence ideas include skill source authentication, runtime behavioural anomaly detection, and formal‑verification‑based rule review.

---

## Practical Applications

### Recommendations for Security Practitioners

1. **Skill source management**: Enterprises should enforce strict skill intake procedures, prioritising officially certified or security‑audited sources over open repositories.

2. **Runtime behavioural baselining**: Establish a normal behavioural baseline for each skill and monitor deviations—although ElasticBack evades simplistic behavioural checks, fine‑grained anomaly detection may still offer some visibility.

3. **Dual‑layer inspection**: Given the fragility of single‑layer defences, adopt a **defence‑in‑depth strategy** combining static inspection, dynamic monitoring, and manual spot‑checks.

4. **Principle of least privilege**: Restrict the permission scope of agent skills, so that even if a backdoor is triggered, the damage is contained.

### Insights for Researchers

1. **Supply chain security**: ElasticBack opens a new attack surface in LLM supply chain security—the skill ecosystem deserves more research attention.

2. **Defence R&D**: The failure of existing defences highlights the need to develop **detection techniques specifically for conditional backdoors**, rather than simply repurposing traditional security methods.

3. **Red‑teaming**: Before deploying agent skills, it is advisable to conduct red‑team exercises—simulating ElasticBack‑style attacks to assess the system’s actual defensive posture.

---

## References

- Original paper: https://arxiv.org/abs/2608.09577
- PDF version: https://arxiv.org/pdf/2608.09577v1
- Skill dataset source: https://clawhub.ai/skills

