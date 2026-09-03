
# TRACE: A Two-Channel Robust Attribution Watermark for LLM-Agent Trajectories

## Paper Highlights

**TRACE** is a two-channel attribution watermark for LLM agent trajectories that simultaneously defends against deletion and rewriting attacks through a **selection channel** and a **tally channel**. It is the first scheme that achieves zero distortion in action selection, self‑synchronisation under deletions, and unconditional invariance under rewriting.

---

## Core Contributions

### Problem Definition

LLM agents are often delivered to end‑users via resellers. A reseller may repackage the developer’s agent under its own brand, or replace the underlying model with a cheaper one. When ownership is disputed, the only available evidence is the **trajectory log** – the sequence of tool calls, observations, and executed actions – not the model’s internal reasoning. The reseller has full read/write access to this evidence, and existing agent watermarking schemes cannot survive such a threat model.

Specifically, the reseller can attack the watermark in two ways:

- **Deletion attacks** – remove records (e.g., observations) from the log, causing position‑dependent watermarks to lose synchronisation.
- **Rewriting attacks** – rewrite observation text, rename tools, etc., altering the content while preserving the log skeleton.

These attacks impose conflicting requirements: deletion attacks demand a **content‑based** key (position is lost), while rewriting attacks demand a **position‑based** key (content can be arbitrarily changed). No single key can satisfy both.

### Innovative Approach

TRACE’s core insight is that a trajectory has enough capacity for **two separate watermarks**.

**Selection Channel**:
- **Carrier**: the specific action chosen in each decision group.
- **Key source**: local context of previous content `ctxi := enc(Ai-1) || key1`.
- **Implementation**: an **exponential race** that samples from the agent’s original distribution without distortion – each candidate action gets a key‑driven random clock, and the first to fire is selected.
- **Key property**: the sampling distribution is **exactly** the agent’s original distribution (Theorem 5.1), so task success rate is unaffected; deletions affect only the adjacent group (blast radius one), and the detector can resynchronise at the next group.

**Tally Channel**:
- **Carrier**: the number of records in each decision group.
- **Key source**: the group’s position in the trajectory skeleton.
- **Implementation**: after each decision group, context‑independent redundant records are appended, the count determined by the positional key.
- **Key property**: both the count and the key are functions of the **trajectory skeleton** – the sequence of decision/observation labels, which rewriting cannot alter – hence this channel is **strictly invariant** under arbitrary rewriting (Theorem 5.3).

The two channels use independent keys, do not interfere, and are designed against deletion and rewriting respectively.

**Theoretical Guarantees**:
- The selection channel’s detection signal scales with decision entropy: each decision contributes at least half its entropy, while deterministic decisions contribute zero (Theorem 5.2).
- To disable both channels, an attacker must corrupt the skeleton and alter at least a constant fraction of group content (Theorem 5.4).

### Results

Evaluated on **ToolBench** and **ALFWorld**:

| Metric | Result |
|--------|--------|
| Task success rate | **On par** with the unwatermarked agent |
| Selection channel detection score (long trajectories) | Near **z = 100** |
| Detectability under 70% deletion | **Remains detectable** |
| Tally channel under LLM rewriting | **Strictly invariant** (any strength) |

TRACE achieves **zero distortion** (no change to the agent’s action distribution) while providing strong attribution – a fundamental distinction from prior work.

### Practical Deployment Potential

**Highly feasible**. TRACE is designed for real‑world commercial scenarios:

- **Reseller scenarios**: any agent distributed through third‑party resellers faces ownership risks; TRACE provides a means of attribution without trusting the reseller.
- **Compliance and auditing**: when agent actions cause harm, TRACE helps determine liability.
- **No inference‑time changes**: watermarking is embedded at the decision layer, not touching internal reasoning, making deployment minimally invasive.
- **Log/execution consistency auditing**: the detector can verify both the reseller‑published log and a reconstructed grouping from execution – discrepancies themselves serve as evidence of tampering.

---

## Technical Details

### 1. Trajectory Modelling

An agent trajectory is defined as a labelled finite sequence `τ := (e₁, ..., e_T)`, where each record has a role label `ρ(e_t) ∈ {dec, obs}`. The **skeleton** `s(τ) := (ρ(e₁), ..., ρ(e_T))` is the label sequence itself. Rewriting attacks do not change the skeleton.

**Decision‑group segmentation**: the trajectory is partitioned into consecutive blocks `g₁, ..., g_m`, each containing exactly one `dec` record followed by all subsequent `obs` records until the next `dec`. Let `k_i` be the number of `obs` records in group `i`.

### 2. Selection Channel: Exponential Race

**Context construction**:
```
ctx_i := enc(A_{i-1}) || key₁
ctx₁ := bootstrap || key₁
```
where `enc` is a length‑preserving encoding of action identities.

**Candidate scoring**:
```
r_b := DRBG(key = H(ctx_i), nonce = b) ∈ (0,1),  b ∈ B_i
```
Each candidate `b` gets a pseudo‑random value driven by the key.

**Selection rule (exponential race)**:
```
b_i := argmin_{b∈B_i} (-ln r_b) / P_i[b]
    = argmax_{b∈B_i} r_b^{1/P_i[b]}
```
Each candidate runs its own key‑driven clock; the agent selects the one that fires first.

**Detection statistic**:
```
φ_i := -ln(1 - r_{b_i})
```
Under the null (unwatermarked), `φ_i ~ Exp(1)`. Aggregate over valid groups:
```
X₁ := Σᵢ φᵢ,  z₁ := (X₁ - n) / √n
```
`z₁` follows an upper‑Gamma tail, used for hypothesis testing.

### 3. Tally Channel: Skeleton‑Based Redundant Records

The tally channel’s key and counts are both functions of the skeleton `s(τ)`. Since rewriting does not change the skeleton, this channel is strictly invariant under rewriting.

### 4. Pseudorandom Foundation

Both channels share a deterministic RNG built on **HMAC‑SHA512**. The embedder and detector recompute the same values from the same inputs without transmitting any side information.

---

## Experimental Setup

### Threat Model

**Provider (defender)**:
- Holds keys `(key₁, key₂)`
- Embeds TRACE watermark at decision time
- Goal: determine whether a trajectory was generated by its agent at a pre‑set false positive rate

**Reseller (attacker)**:
- **No key** – cannot forge, only attempts to remove
- **No re‑execution** – can only edit the already‑logged trajectory
- **Utility and consistency constraints** – edited logs must remain plausible and consistent

### Evaluation Settings

| Item | Details |
|------|---------|
| Benchmarks | ToolBench, ALFWorld |
| Baselines | Red‑Green, Multi‑bit schemes |
| Deletion attack | Each `obs` independently deleted with probability `r`, scanning `r` from 0 to 0.7 |
| Rewriting attack | **LLM Rewriter** – a novel informed, fidelity‑preserving rewriting attack; the LLM returns a plausible alternative action after seeing the true choice |
| Rewriting strength | `q` from 0 to 1 |

---

## Analysis

### Theoretical Depth

TRACE provides **structural theoretical guarantees**, unlike most prior work that relies on empirical robustness:

- **Exact null laws**: the null distributions of both detectors are exact, not asymptotic approximations.
- **Entropy lower bound**: the cost of distortion‑free detectability is quantified – each decision contributes at least half its entropy; deterministic decisions (entropy 0) cannot carry signal.
- **Joint erasure theorem**: any attack that silences both channels must edit the skeleton, altering at least a constant fraction of group content.

### Comparison with Prior Work

| Scheme | Zero‑distortion | Deletion‑robust | Rewriting‑robust |
|--------|----------------|----------------|------------------|
| Agent Guide | ✗ | Not proven | Not proven |
| AgentMark | ✓ | Not proven | Not proven |
| AgentWM | ✗ | Not proven | Not proven |
| ActHook | ✗ | Not proven | Not proven |
| **TRACE** | **✓** | **✓** | **✓** |

TRACE’s key differentiator is its threat model: **the attacker is the party holding the evidence**. Traditional log signing can prove integrity, but not attribution against an edited log – the reseller simply discards the provider’s signature. TRACE’s behavioural watermarking provides a complementary signal that remains valid even when the record‑holder is adversarial.

### Design Philosophy: Complementary Embedding

The most elegant aspect of TRACE is the **complementarity** of its two channels:
- The selection channel’s content‑key makes it self‑synchronising under deletion, but fragile under rewriting.
- The tally channel’s position‑key makes it invariant under rewriting, but fragile under deletion.
- Combined, any single attack can only disable one channel; disabling both requires an extremely high cost.

---

## Practical Applications

### 1. Agent Service Reselling

If your agent is distributed via third‑party platforms (API marketplaces, cloud providers), TRACE enables attribution without trusting the reseller:
- Watermark all outgoing trajectories server‑side.
- When a reseller claims a trajectory comes from its own system, you can verify the true source via the detector.

### 2. Compliance Auditing

For regulated agent applications (finance, healthcare, security):
- Auditors can extract watermarks from logs post‑facto to confirm that actions originated from the claimed agent.
- Consistency checks between logs and execution can detect reseller tampering.

### 3. Deployment Considerations

- **Key management**: use independent keys for the two channels; consider storing them in separate security domains.
- **Entropy awareness**: in scenarios with many deterministic decisions (e.g., rule‑driven agents), the selection channel’s signal weakens – the tally channel provides complementary protection.
- **Short trajectories**: the paper handles short logs via explicit pooling rates.
- **No reliance on inference traces**: TRACE only reads and marks execution logs (tool calls, observations, actions), not internal reasoning that may be hidden or encrypted.

---

## References

- Original paper: [Trace: A Two-Channel Robust Attribution Watermark for LLM-Agent Trajectories](https://arxiv.org/html/2607.08400v1)
