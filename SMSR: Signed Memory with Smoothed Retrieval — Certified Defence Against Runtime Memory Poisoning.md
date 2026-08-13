
# SMSR: Signed Memory with Smoothed Retrieval – Certified Defence Against Runtime Memory Poisoning in Persistent LLM Agent Systems

[![arXiv](https://img.shields.io/badge/arXiv-2606.12703-b31b1b.svg)](https://arxiv.org/abs/2606.12703) [![GitHub](https://img.shields.io/badge/GitHub-Repository-181717)](https://github.com/tarun-ks/smsr)

## Summary

This paper presents the first certified defence against Multi‑Session Memory Poisoning (MSMP) attacks. SMSR combines **HMAC‑SHA256 write‑time provenance** with **randomised memory ablation + verdict‑based majority voting**. Across 3,150 repeated trials in 15 enterprise scenarios, the framework reduces the attack success rate (ASR) of **unsigned** adversaries from 93–100% to **0%**, and bounds the ASR of **certified** (authenticated) attackers below theoretical certificates – down to **8.0%** (95% CI [5.8%, 10.9%]) for a single injection.

---

## Core Research

### Problem Definition
Retrieval‑augmented generation (RAG) agents are increasingly adopting **persistent cross‑session memory** – systems accumulate interaction logs, retrieved documents, and inferred facts across user sessions. This creates a new attack surface: an adversary can inject carefully crafted memories through normal interaction channels (as an employee, customer, or via a compromised upstream tool). Once retrieved, these memories influence responses to *all future users* – without touching model weights or application code.

The authors formalise this as **Multi‑Session Memory Poisoning (MSMP)**. Prior attacks such as MINJA achieve ~76‑99% ASR, and AgentPoison reaches 62% end‑to‑end ASR with only one poisoned entry in a database of 23,000 records. However, existing defences offer no formal guarantees:

- **Static corpus defences** (RobustRAG, ReliabilityRAG) assume a fixed knowledge base indexed offline, and cannot handle dynamic memory writes.
- **Heuristic filters** (keyword blacklists, perplexity, semantic anomaly) are completely bypassable by fluent enterprise‑style text.

### Innovative Approach
SMSR adopts a **dual‑component architecture**:

**Component 1: HMAC‑SHA256 Write‑Time Provenance**  
Every legitimate memory write must be signed by a trusted service oracle using a server key *K*:  
`τᵢ = HMAC_K(contentᵢ || session_idᵢ || timestampᵢ)`.  
During retrieval, only entries with valid signatures enter the candidate pool. This completely blocks unsigned adversaries – no matter how carefully they craft content.

**Component 2: Randomised Memory Ablation + Verdict‑based Majority Voting**  
For *authenticated* adversaries who can pass Component 1 (i.e., legitimate users), this layer provides a second line of defence:

1. **Over‑fetch**: Retrieve top‑*m* (*m* > *k*) verified candidates for query *q*.
2. **Random ablation**: Run *n_runs* independent trials; in each, uniformly sample *k* out of *m* candidates without replacement.
3. **Verdict aggregation**: Each trial generates an LLM response, then a judge model (LLM Judge) assigns a verdict (*correct / malicious / neither*). The final output is decided by **majority vote of these verdicts**.

**Key Theoretical Contributions**

- **Impossibility Theorem**: Proves that *any* filter relying solely on retrieval‑time content inspection, without write‑time provenance, cannot provide a non‑trivial security certificate against adaptive MSMP attackers. This means **provenance is necessary**.
- **Hypergeometric Certificate**: Derives a formal bound for Component 2 – when the candidate pool contains *t′* adversarial entries, the probability that the majority verdict is malicious is at most `δ(t′, m, k, n_runs)`.
- **Consistent Minority Effect (CME)**: Formalises a vulnerability of string‑level majority voting – attackers win because their responses are textually consistent, even when they are a numerical minority. Verdict‑based aggregation is inherently immune.

### Research Results

**Experimental Setup** – Simulated the Nexora Corp enterprise RAG agent across 15 business scenarios (finance, compliance, IT security, HR, procurement, incident response). Memory stores pre‑populated with 10 signed legitimate policy entries. Key parameters: *m*=20, *k*=5, *n_runs*=5.

**Key Findings**

1. **Component 1 is perfectly effective against unsigned attacks**: Across all unsigned injection variants (direct, flood, subtle, and bypass designed to evade heuristics), ASR dropped from 93–100% to **0%**. Heuristic defences failed completely (100% ASR) under bypass attacks.

2. **Component 2 provides provable bounds for authenticated adversaries**: In production‑scale storage (*m*=20, *t*=1), ASR dropped from 93–100% to **8.0%** (95% CI [5.8%, 10.9%], *n*=450), below the worst‑case bound δ=10.4%. Increasing *n_runs* to 7 tightens the bound to 7.1%.

3. **Certificate tightness**: In small‑store evaluations (*m′* = 10 + *t*), direct injection ASR for *t*=1 was 37.8% (95% CI [33.4%, 42.3%]), below the bound δ=41.5%. Flood variants’ ASR lay near the bound (CI crossing δ), confirming tightness.

4. **Judge reliability**: Agreement between Haiku and Sonnet judges reached κ=0.955 (*n*=84), with 97.6% raw consistency.

5. **End‑to‑end query‑only attacks**: When the agent itself writes poisoned memories via queries (rather than pre‑seeding), SMSR reduced ASR from 65.3% to **5.3%** (*n*=150, non‑overlapping CIs).

6. **Clean‑query utility**: 90% under Component 1, 85% with the full defence.

### Real‑World Applicability

SMSR is designed for practical enterprise constraints:

- **Compatible with existing RAG architectures**: Only the write path (signing oracle) and query path (ablation + aggregation) need changes – no LLM retraining.
- **No re‑training required**: All defences operate at inference time.
- **Key management**: The server key *K* must be stored in a key manager or HSM, never in application code.
- **Open‑source code and data**: https://github.com/tarun-ks/smsr

---

## Technical Details

### SMSR Protocol (Pseudo‑code)

```pseudocode
Algorithm 1: SMSR – Signed Memory with Smoothed Retrieval

Input: query q, memory store M, secret key K, parameters m, k, n_runs, judge J

// Component 1: retrieve only verified candidates
C ← RETRIEVE(q, {m_i ∈ M : VERIFY_K(m_i) = 1}, m)

// Component 2: randomised ablation + verdict aggregation
V ← []
for j = 1 to n_runs do
    C^(j) ← sample k candidates uniformly from C without replacement
    r^(j) ← LLM(q, C^(j))
    V.append( (J(r^(j), q), r^(j)) )

v_hat ← arg max_v Σ_j 1[V_j.verdict = v]
return first response r^(j) whose verdict == v_hat
```

### Component 2 Certificate Bound

Let `M_verified` contain `N` entries, of which at most `t′` are adversarial. Define:

```
p_clean = C(m - t′, k) / C(m, k)
```

the probability that a single ablated run contains no adversarial entries. Then the probability that the majority verdict is malicious is at most:

```
δ(t′, m, k, n_runs) = Σ_{i=⌈n_runs/2⌉}^{n_runs} C(n_runs, i) (1 - p_clean)^i · p_clean^{n_runs - i}
```

**Intuition**: The attacker wins only if more than half of the runs include adversarial content *and* each such run yields a malicious response. The certificate conservatively upper‑bounds the latter probability by 1, giving a provable worst‑case guarantee.

### Consistent Minority Effect (CME) Quantified

String‑level majority voting fails because adversarial responses are textually similar (the malicious fact is specific and repeatable), while legitimate responses vary in phrasing (“I don’t know” has many paraphrases). If legitimate responses come from a distribution with entropy `H_clean`, the probability that any legitimate string appears more than once is `O(n_clean² · 2^{−H_clean})`. In natural language, `H_clean` is large – so an attacker with even one run can become the most frequent string. Empirically, string‑level voting gave 93.3% ASR on the same ablated runs where verdict‑based aggregation gave only 13.3%.

---

## Experimental Setup

| Item | Specification |
|------|---------------|
| LLMs (Agent + Judge) | Claude Haiku 4.5 (main); Claude Sonnet 4.6 (cross‑model validation) |
| Embedding model | Standard RAG embedder (known to attackers) |
| Signature algorithm | HMAC‑SHA256 (256‑bit output) |
| Key storage | Server‑side key manager or HSM |
| Total trials | 15 scenarios × multiple configurations × 30 repetitions = 3,150 trials |
| Production‑scale trials | 450 independent runs |

**Threat Models** – Two adversary types:

- **Unsigned adversary (A_U)**: Has direct write access to `M` (e.g., via database misconfiguration, SQL injection, or stolen backup). Cannot modify or delete existing entries.
- **Authenticated adversary (A_P)**: A legitimate user of the system, can write entries through normal interaction channels. Injects at most `t` crafted entries per attack. Does not know the server key `K`.

Both adversaries know the embedding model, agent architecture, and retrieval mechanism.

**Security Definition** – A system is `(t, δ)-SMSR‑secure` if for any MSMP attacker injecting at most `t` adversarial entries, the probability that the agent’s response is malicious is at most `δ`.

---

## Analysis & Insights

### Why This Matters
Persistent memory is becoming the standard architecture for enterprise RAG agents – frameworks like LangGraph, AutoGen, and MemGPT all support cross‑session memory. Yet this design creates a fundamental security tension: **the write path is both the source of utility (accumulating useful context) and the attacker’s injection channel**. In multi‑tenant deployments, one user’s interactions can affect all future users’ responses – meaning a single injection can persistently harm the entire system.

While prior work has focused on inversion attacks and embedding vulnerabilities, formal defences against poisoning have been scarce. SMSR fills this gap.

### Theoretical Elegance
The dual‑component design is particularly insightful:

- **Component 1 answers “who wrote this?”** – using cryptographic provenance to cut off unsigned injections.
- **Component 2 answers “how much influence does what was written have?”** – even if the writer is legitimate, the impact of malicious content on system outputs is probabilistically bounded.

The **Impossibility Theorem** is a key contribution: it proves that any content‑only filter is fundamentally futile because the embedding space is dense – for any target query, an attacker can craft fluent, semantically relevant text that passes any content‑based threshold. Thus, provenance is *necessary*.

### Practical Significance of the Certificate
Component 2’s bound is worst‑case (assuming every contaminated run yields a malicious response), yet experiments show actual ASR is often lower. This gives system administrators a **conservative but provable** guarantee – they can compute the worst‑case ASR before deployment and tune *m* and *n_runs* to meet their security goals.

### Limitations and Future Work

- **Key rotation**: Rotating the HMAC key invalidates all previously signed memories, posing operational challenges for long‑running systems.
- **Judge dependency**: The defence relies on a reliable LLM Judge. While inter‑judge agreement is high (κ=0.955), the robustness of the judge itself requires ongoing monitoring.
- **Scaling with *t***: As the number of injections *t* grows, the certificate bound loosens quickly (*t*=2 → δ≈0.402; *t*=3 → δ≈0.684). In practice, this suggests that early detection of large‑scale injection campaigns remains important.

---

## Practical Deployment

### Suitable Scenarios

- **Multi‑tenant RAG agents** sharing a common memory store across users.
- **Compliance‑sensitive applications** (e.g., financial approvals, data retention policies, access control) where erroneous interpretations can cause regulatory or financial harm.
- **Long‑running customer‑service / assistant systems** where memory accumulates across sessions and attackers have a persistent window of opportunity.

### Deployment Recommendations

1. **Prioritise key management** – Ensure reliable HSM or cloud KMS before deploying Component 1; plan key rotation policies.
2. **Tune parameters** – Increase *m* and *n_runs* to tighten the certificate, at the cost of inference latency and cost.
3. **Choose judge diversity** – Use a judge from a different model family or size than the main agent to avoid common‑mode failures.
4. **Monitor abnormal write patterns** – Even with SMSR, monitor for unusual memory writes – when *t* is large, the bound is wider, so early detection is critical.
5. **Incremental rollout** – Deploy Component 1 first (no LLM cost increase) to block all unsigned attacks; enable Component 2 later based on threat landscape.

### Code & Resources

- **GitHub Repository**: [https://github.com/tarun-ks/smsr](https://github.com/tarun-ks/smsr)
- **Original Paper**: [arXiv:2606.12703](https://arxiv.org/abs/2606.12703)

---

## References

- Sharma, T. "SMSR: Signed Memory with Smoothed Retrieval — Certified Defence Against Runtime Memory Poisoning in Persistent LLM Agent Systems." arXiv:2606.12703, 2026. [https://arxiv.org/abs/2606.12703](https://arxiv.org/abs/2606.12703)
