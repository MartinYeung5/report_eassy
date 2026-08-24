
# ChainWatch: A Kill Chain-Aligned Sequential Detection Framework for Multi‑Step Attacks in MCP‑Based AI Agent Systems

## Paper Summary

ChainWatch is a detection framework that addresses multi‑step attacks in AI agent systems built on the Model Context Protocol (MCP). Unlike existing per‑call defences, it analyses whole sequences of tool invocations by mapping them to a six‑phase kill chain and applying a Hidden Markov Model (HMM) for state inference. The work shows that individually benign calls can form malicious chains—with attack success rates exceeding 90% on GPT‑4.1 in unprotected settings—and that ChainWatch can successfully catch all five tested attack scenarios that bypass single‑call filters.

---

## Core Contributions

### Problem Definition

MCP (introduced by Anthropic in November 2024) enables AI agents to connect to external tools, databases, and services. This introduces a critical security gap: attackers can combine individually harmless tool calls into a malicious sequence that evades per‑request checks. In unprotected systems, such chained attacks succeed at over 90% against GPT‑4.1. STAC research shows that 21 out of 36 documented AI agent attacks span four or more stages. While defences like MCPShield, MCP‑Guard, and MindGuard exist, they focus on single‑call or single‑server threat models and lack session‑level sequence modelling.

### Novel Approach

1. **Six‑Phase MCP Kill Chain**  
   The framework abstracts multi‑step MCP attacks into six observable phases: **Reconnaissance**, **Trust Building**, **Injection**, **Escalation**, **Lateral Movement**, and **Exfiltration**. This extends the existing Promptware kill chain with two MCP‑specific early phases.

2. **HMM‑Based Phase Classifier**  
   Phase assignment is treated as a hidden state inference problem. A 20‑dimensional feature vector per tool call is fed into an HMM that estimates the most likely phase sequence. The transition matrix is constrained with three rules: forward transitions are preferred over backward, jumps of more than two phases are penalised, and a small backward probability is retained to handle repeated early‑stage actions.

3. **Five Session‑Level Detection Rules**  
   Using a sliding window of length *k*=10, the framework inspects the stream of inferred phases and triggers alerts for patterns such as sensitive data access after reconnaissance, cross‑server access, data exfiltration after reads, rapid kill‑chain acceleration, and late‑stage configuration writes.

### Results

Five attack scenarios from the security literature were validated (covering Direct Sequential Attacks, Indirect Injection Chains, and Hybrid Multi‑Stage Attacks):

- **Financial fraud (DSA)**: `get_balance` → `list_payees` → `add_payee` → `transfer_funds` → triggers R4 and R3 (CRITICAL).
- **GitHub data theft (IIC)**: malicious injection via GitHub issue → triggers R3 (CRITICAL).
- **WhatsApp rug‑pull (HMSA)**: tool definition silently replaced after approval → triggers R4 (CRITICAL).
- **Cross‑agent privilege escalation (HMSA)**: persistence via config write → triggers R5 and blocks the call.
- **Credential harvesting (DSA)**: `list_tools` → `read_env` → `read_ssh_config` → `post_to_webhook` → triggers R1 and R3 (CRITICAL).

All five chains bypass per‑call defences but are detected by ChainWatch.

### Practical Deployment Potential

ChainWatch is deployed as a transparent proxy between MCP clients and servers, requiring no modifications to models, hosts, or servers. It is designed to work alongside—not replace—existing per‑call defences. Key parameters (window size *k*=10, step threshold *m*=5) are configurable to balance false positives in different environments.

---

## Technical Details

### 20‑Dimensional Feature Extraction

Each raw MCP tool call is converted into a fixed 20‑dimension vector grouped into five categories:

| Group | Dimensions | Description |
|-------|------------|-------------|
| Tool Category (TC) | 5 | One‑hot: READ, WRITE, EXECUTE, NETWORK, CONFIGURE |
| Parameter Sensitivity (PS) | 1 | Weighted sum of credentials, paths, URLs, encoded data |
| Data Flow (DF) | 4 | Binary flags: internal read, external write, chained pass, cross‑server |
| Temporal Features (TF) | 3 | Inter‑call interval, call rate over *k*, session duration |
| Output Characteristics (OC) | 7 | Instructional text, XML tags, description‑output mismatch, size anomaly, hash change, encoded data, external URL |

Final feature vector: **v = [TC, PS, DF, TF, OC] ∈ ℝ²⁰**.

### HMM Formulation

The HMM is defined as **λ = (S, Σ, A, B, π)**:

- **S**: set of six kill‑chain phases (hidden states)
- **Σ**: observation space of 20‑dim feature vectors
- **A**: state transition matrix (with designed constraints)
- **B**: emission distribution (continuous)
- **π**: initial state distribution

This builds on prior work by Holgado et al., which used HMM and Viterbi decoding for APT kill‑chain prediction, but replaces discrete IDS alerts with continuous feature vectors.

### Five Detection Rules

| Rule | Trigger Condition | Severity |
|------|-------------------|----------|
| R1 | Reconnaissance followed by sensitive data access | WARNING |
| R2 | Access to ≥2 servers with sensitive‑data flow flag active | WARNING |
| R3 | High‑phase READ then, within *m* steps, a NETWORK call carrying that data | **CRITICAL** |
| R4 | Kill‑chain acceleration (skipping two or more phases) | WARNING |
| R5 | Configuration write at phase 4 or higher | **CRITICAL** |

Rules R3 and R5 trigger CRITICAL alerts and block pending calls; R1, R2, and R4 raise WARNING alerts for human review.

---

## Research Setup

### Deployment Architecture

ChainWatch sits as a transparent proxy between the MCP client and MCP servers. The client and host application are trusted; MCP servers are treated as potentially adversarial. The proxy runs as a non‑public monitoring layer—attackers cannot know its thresholds or window parameters.

### Threat Model

**Assumed attacker capabilities**: register/compromise MCP servers to control visible tools; embed adversarial content in tool descriptions and outputs; replace tool definitions after user approval; coordinate steps across multiple servers.

**Assumed limitations**: cannot access the MCP client internals, alter the underlying model behaviour, or learn ChainWatch’s detection configuration.

### Parameters to Validate

The paper notes that exact HMM transition probabilities and detection thresholds require empirical calibration (e.g., via Baum‑Welch) using real MCP trajectories. The R2 rule may produce false positives in legitimate multi‑service workflows and needs tuning per deployment. MCP‑SafetyBench is suggested as a starting point for empirical validation.

---

## Critical Analysis

### Methodological Contribution

ChainWatch’s main contribution is transplanting well‑established “sequence detection” techniques from traditional network security into the nascent field of AI agent security. Prior HMM‑based multi‑step attack detection in IDS is adapted by redefining attack phases at the *semantic level of tool calls*. The six‑phase kill chain provides this abstraction, enabling a proven tool (HMM) to work in a new domain.

### Design Rationale

- Window size *k*=10 is chosen based on documented attacks spanning 4–7 calls, with margin.
- Step threshold *m*=5 triggers within half a window after the first suspicious signal.
- Transition constraints (forward preference, limited jumps, small backward probability) reflect reasonable priors on real attack behaviour.
- The 20‑dim feature design covers call‑level attributes (category, sensitivity), inter‑call context (data flow, timing), and server‑side corruption signals (output anomalies), providing a layered representation.

### Limitations

- **Lack of empirical validation**: the work is a design specification, not an implemented system; HMM probabilities are design choices, not estimated from real data.
- **Dataset gap**: existing MCP security benchmarks (MCP‑Tox, MCP‑AttackBench) are per‑call and lack chained sequences—this is a community‑wide issue.
- **Evasion risk**: a sophisticated adversary might blur phase boundaries (e.g., intersperse benign calls in reconnaissance) or insert noise to evade R2. Robustness against such “boundary‑fuzzy” attacks remains untested.

### Relationship to Existing Work

ChainWatch is not a replacement but a **complement** to MCPShield, MCP‑Guard, and MindGuard:
- MCPShield accumulates historical trust per server.
- MCP‑Guard applies cascaded per‑call filtering.
- MindGuard inspects LLM internal attention patterns for poisoned metadata.
- ChainWatch fills the session‑level blind spot of these point‑defences.

---

## Practical Recommendations

### For AI Agent Platform Developers

1. **Adopt layered defence** – combine ChainWatch‑style sequence detection with per‑call filters (input sanitisation, tool definition monitoring, behaviour baselines).
2. **Mandate session‑level logging** – MCP does not require audit logs; deploy a proxy layer that records full tool‑call sessions as a prerequisite for sequence analysis.
3. **Enforce multi‑step verification** – for sensitive operations (transfers, credential reads, config writes), require contextual validation across multiple phases (e.g., `transfer_funds` should not follow immediately after `add_payee`).

### For Security Researchers

1. **Build chained attack benchmarks** – extending MCP‑SafetyBench with multi‑step scenarios is the critical bottleneck for moving from design to validation.
2. **Explore alternative sequence models** – HMM assumes independent observations and first‑order Markov transitions; GNNs, transformers, or temporal logic may better capture complex patterns.
3. **Conduct red‑team evaluations** – test framework robustness against evasion techniques like phase obfuscation and noise injection before deployment.

### Deployment Considerations

- **Performance overhead**: evaluate latency introduced by 20‑dim feature extraction and HMM inference, especially in high‑throughput settings.
- **False‑positive management**: R2 may frequently trigger in legitimate cross‑service workflows; establish graded alerts and human review procedures.
- **Transparency trade‑off**: running ChainWatch as a non‑public monitor improves detection but requires dedicated system administration for maintenance and tuning.

---

## References

- Original paper: [ChainWatch: A Kill Chain-Aligned Sequential Detection Framework for Multi-Step Attacks in MCP-Based AI Agent Systems](https://arxiv.org/abs/2607.19432)
- PDF: [https://arxiv.org/pdf/2607.19432](https://arxiv.org/pdf/2607.19432)
