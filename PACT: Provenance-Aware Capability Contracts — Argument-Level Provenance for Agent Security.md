
# PACT: Provenance-Aware Capability Contracts — Argument‑Level Provenance for Agent Security (2026)

## Paper Highlights

PACT is a runtime monitoring mechanism that shifts the security boundary from the whole‑tool‑call level down to the **argument level**. By assigning semantic roles to each tool parameter and tracking cross‑step value provenance, it resolves the fundamental security‑utility dilemma in mixed‑trust LLM agent scenarios. The core insight: indirect prompt injection is dangerous not because untrusted content appears in the context, but because it can bind to *authority‑carrying parameters* (e.g., recipients, URLs, commands). PACT achieves **100% security and 100% utility** at the mechanistic level, and in full deployments it recovers 38.1%–46.4% utility while maintaining 100% security on the strongest models.

---

## Core Research Content

### Problem Definition

Tool‑calling LLM agents need to read from untrusted web pages, emails, files, and API outputs while simultaneously invoking privileged tools. Existing defenses make trust decisions at the granularity of an entire tool call—creating a fundamental **granularity mismatch**. If we block any call influenced by external content, many legitimate "read‑then‑act" workflows break; if we allow all, attackers can hijack target addresses or commands via injected instructions. Take `send_email(recipient, body)` as an example: web content *should* determine the email body, but it must *never* determine the recipient—yet whole‑call policies cannot express this distinction.

### Innovative Approach

PACT's key innovation is pushing the security boundary down to the **argument level**. It is built on three layers:

1. **Argument‑Level Contracts** – Each tool parameter is assigned a semantic role (e.g., `target`, `command`, `credential`, `content`) and a minimum trust level required for that role. Contracts are defined at four precision levels L0–L3, from "opaque blocking" to "certified routing".

2. **Cross‑Step Provenance Tracking** – Every runtime value carries a provenance tag `π(v) = ⟨O(v), τ(v), B(v)⟩`, recording its origin set, current trust level, and unresolved obligations. When values are merged, provenance conservatively combines: `merge = ⟨O₁∪O₂, min(τ₁,τ₂), B₁∪B₂⟩` — trust is never automatically promoted.

3. **Certified Routing (L3)** – Allows a trusted "certification process" to unblock a parameter within a limited scope, while preserving the original source and ensuring that certificates cannot be reused across roles.

Before each tool call, the PACT monitor checks that every argument's provenance satisfies its contract.

### Research Results

**Mechanistic Validation (Oracle Provenance)** – On 17 mixed‑trust diagnostic scenarios, PACT L2 achieved **100% utility and 100% security** with zero false positives and zero false negatives. Baselines: Vanilla (100% utility, 0% security), CaMeL (100% security, only 33.3% utility), and FIDES (neither perfect).

**Full AgentDojo Deployment** – Over five models, PACT reached **100% security** on the three strongest models, recovering **38.1%–46.4% utility** — outperforming CaMeL by **8–16 percentage points** at the same security level (see Table 2 in the paper).

**Ablation Studies** – Removing semantic roles dropped utility by 60 percentage points; removing cross‑step provenance created security holes. Simple per‑argument checking without role distinction could not match PACT's combined safety and utility.

### Practical Deployment Potential

- **No model modifications** – PACT is a runtime monitor, independent of the model's adversarial robustness, and can be plugged into existing agent frameworks.
- **Automated contract synthesis** – The paper presents a pipeline that synthesises contracts from tool schemas, achieving 87.1% role accuracy and 77.4% provenance accuracy on 20 real MCP tools.
- **Wide applicability** – Suitable for mail agents, browser agents, file operation agents, API orchestrators, and any LLM application that calls tools.
- **Low runtime overhead** – Online checking is linear in the number of parameters, involving only lattice comparisons, set intersections, and certificate scope checks.

---

## Technical Details

### Contract Formalisation

For a tool `t`, PACT defines a contract:

`C_t = (ℓ, {a_i}_{i=1}^k, o)`

where `ℓ ∈ {L0, L1, L2, L3}` is the precision level and `o` is the output provenance specification.

Each argument entry is:

`a_i = (name_i, role_i, τ_i^min, F_i, R_i, D_i)`

where `role_i` is the semantic role, `τ_i^min` is the minimum trust level, `F_i` is the set of forbidden origins, `R_i` is the set of obligations, and `D_i` is the set of allowed certification processes for L3.

### Trust Lattice

Trust values form an ordered lattice:

`TRUSTED > USER > TOOL_OUTPUT > EXTERNAL`

User instructions get `USER` trust, trusted constants get `TRUSTED`, and tool outputs are assigned according to the `OutputSpec`.

### Runtime Check Algorithm

Before executing `t(v₁, …, vₖ)`, the monitor checks each argument `vᵢ` against:

- `τ(vᵢ) ≥ τᵢ^min` (trust level sufficient)
- `O(vᵢ) ∩ Fᵢ = ∅` (origin not in forbidden set)
- `B(vᵢ) ∪ Rᵢ ⊆ Discharged` (obligations fulfilled)

At L3, a failing check may be rescued via a certification process, provided that the process covers the failed predicate and is applicable to that argument role.

### Certification Certificates

A certification process returns a scope‑limited certificate:

`d = (r, s, ρ, τ^max)`

where `r` is the resolved failed predicate, `s` the certification process, `ρ` the role scope, and `τ^max` the effective trust ceiling. Applying the certificate produces a derived value; the original source is preserved and only obligations within the scope are discharged.

---

## Experimental Setup

### Environment

- **Benchmark**: AgentDojo v1 full runtime with 97 benign user tasks and 27 injection tasks across Banking, Workspace, Slack, and Travel scenarios.
- **Models**: Owen‑turbo, Owen‑plus, Owen‑max, Owen2.5‑72B, and GPT‑4o‑mini (three model families).
- **Baselines**: Vanilla (no defence), FIDES, and CaMeL.

### Contract & Provenance Inference Pipeline

- Contracts are automatically synthesised from tool schemas (name, parameter names, type annotations, natural‑language descriptions).
- Deterministic rules map `recipients`, `URLs`, `endpoints` → `target`; `shell commands`, `executable queries` → `command`; `API keys`, `tokens`, `passwords` → `credential`; `message bodies`, `summaries` → `content`.
- Provenance is inferred via exact structural matching, role‑aware heuristics, and an LLM‑based classifier.
- The pipeline synthesised 74 contracts, achieving 87.1% role accuracy and 77.4% provenance accuracy on 20 real MCP tools.

---

## Comprehensive Analysis

### Theoretical Contribution: Reframing the Problem

PACT's most profound contribution is reframing indirect prompt injection: instead of asking "how to detect or filter malicious content", it asks "whatever the content, what is it *allowed to decide*?" This is a paradigm shift from content detection to permission binding.

This shift acknowledges that agents *must* consume untrusted content, and avoids relying on the model to ignore all malicious text. The same web page can determine the email body but never the recipient – PACT expresses this semantic distinction structurally.

### Design Elegance

Several thoughtful trade‑offs stand out:

- **Layered contracts (L0–L3)** – Allows coarse‑grained blocking (L0) and progressive refinement to L3, supporting a "security‑first" deployment while leaving room for utility optimisation.
- **Conservative provenance merging** – `min(τ₁,τ₂)` ensures trust is never silently promoted via ordinary data flow – a critical safety property.
- **Certified routing, not trust upgrade** – L3 does not simply "turn" untrusted data into trusted; it unblocks within a limited scope while keeping the original source, preserving audit integrity.

### Boundaries of Formal Guarantees

Three theorems provide formal assurance:

- **Theorem 3** (separation from whole‑call monitoring): In mixed‑trust scenarios, argument‑level contracts achieve full utility and security, while any whole‑call monitor must produce false positives or false negatives.
- **Theorem 4** (monotonicity of safe refinements): Refinements from L0 to L3 preserve security while monotonically reducing unnecessary blocks.
- **Theorem 5** (prefix‑based reliability): Each decision relies only on accumulated provenance, not on future actions.

These theorems assume conservative provenance propagation and correctly specified contracts – the paper honestly admits that automated contract synthesis and provenance inference are the main deployment bottlenecks.

### The Deployment Gap

The gap between the oracle mechanism (100% utility + 100% security) and full deployment (38.1%–46.4% utility + 100% security) clearly shows the current technology bottleneck. The authors attribute the gap to *provenance inference and contract synthesis fidelity*, not the runtime policy itself. In other words, PACT has achieved theoretical completeness in *what to block and what to allow*, but the upstream tasks of "knowing the origin of each value" and "knowing the role of each parameter" remain open challenges.

---

## Practical Applications

### Suitable Scenarios

1. **Email agents** – PACT allows web content to populate the email body (`content` role) but blocks it from setting the recipient (`target` role).
2. **Browser agents** – Distinguishes which parameters may be influenced by web pages and which must come from user instructions or trusted constants.
3. **Code‑execution agents** – Prevents file content from determining the command itself (`command` role), while allowing it to determine the command's input data (`content` role).
4. **API orchestrators** – Cross‑step provenance prevents attackers from "washing" untrusted content through multi‑step data flows and binding it to authority parameters.

### Deployment Recommendations

- **Phased rollout** – Start with L0 or L1 (conservative security), then gradually upgrade to L2/L3 as contracts are refined.
- **Contract maintenance** – Automated synthesis accuracy (87.1% role, 77.4% provenance) requires human review and continuous optimisation of the contract library.
- **Complementary defences** – PACT can be combined with content filtering and prompt injection detectors – it addresses permission binding, not content detection.
- **Auditability** – The provenance tagging system naturally supports security auditing: each blocked call carries explicit failure reasons (trust insufficient / origin forbidden / obligation unresolved), easing debugging and policy tuning.

---

## References

- Original paper: [The Granularity Mismatch in Agent Security: Argument‑Level Provenance Solves Enforcement and Isolates the LLM Reasoning Bottleneck](https://arxiv.org/abs/2605.11039) (arXiv:2605.11039, 2026)


*This analysis is based on the original paper; all data and conclusions are quoted from the main text and appendices.*
```
