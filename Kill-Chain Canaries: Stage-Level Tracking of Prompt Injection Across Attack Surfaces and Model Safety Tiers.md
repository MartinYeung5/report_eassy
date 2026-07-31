
# Kill-Chain Canaries: Stage-Level Tracking of Prompt Injection Across Attack Surfaces and Model Safety Tiers

> **Original Paper**: *Kill-Chain Canaries: Stage-Level Tracking of Prompt Injection Across Attack Surfaces and Model Safety Tiers* (2026)  
> **Author**: Haochuan Kevin Wang (MIT)  
> **arXiv**: 2603.28013

---

## Paper Summary

This paper reframes prompt injection evaluation from a **task-level success rate** to a **pipeline-stage localization** problem. The authors introduce a “kill-chain canary” method—a trackable cryptographic token that traverses four critical stages of an LLM agent pipeline—to pinpoint exactly *where* each model’s defenses activate, rather than merely whether an attack ultimately succeeds. Across **5 frontier models, 4 attack surfaces, 5 defense conditions, and 764 runs**, the key counter‑intuitive finding is that **all models are exposed to the injection**; the real security gap lies entirely in downstream propagation and execution.

---

## Core Research Contributions

### Problem Definition

Prompt injection—where attackers embed malicious instructions into data processed by LLM agents—is a central threat in deployed agentic AI. However, existing evaluations report a monolithic **Attack Success Rate (ASR)** at the task level, which conflates two fundamentally different questions:

- Does the model *see* the injection?
- Does the model *execute* the injection?

For example, an agent calls `get_webpage()` to fetch poisoned content, then calls `write_memory()` to summarize it, and finally another agent reads it via `read_memory()`. If the second agent’s ASR is 0%, it could be because the injection was stripped during `write_memory` or because it survived but the agent refused to act. These are architecturally distinct security states, yet traditional metrics cannot distinguish them.

### Innovative Method

**Kill‑Chain Canaries.** The core methodological innovation is embedding a unique cryptographic token (`SECRET-[A-F0-9]{8}`) into every injection payload. This token is tracked by a `PropagationLogger` across four discrete stages:

- **EXPOSED** – token appears in any tool return result.
- **PERSISTED** – token appears in a `write_memory` call.
- **RELAYED** – token is read by Agent B.
- **EXECUTED** – token appears in arguments of any outbound tool call.

By tracking token presence at each stage, researchers can pinpoint *where* defenses activate—not just *whether* they activate.

**Experimental Platform.** The authors built `agent_bench`, a minimal (~600 lines) Python multi‑agent evaluation framework with components: `MemoryStore` (key‑value with `read/write_memory`), `ToolRegistry` (permission‑gated scheduler), `MultiAgentOrchestrator` (two‑agent coordinator), and `PropagationLogger` (JSONL event logging).

**Attack Surfaces.** Four different injection channels:
- **memory_poison** – pre‑seeded via `MemoryStore.seed()`
- **tool_poison** – injection appears in return values of `get_webpage()` or `query_db()`
- **propagation** – two‑agent memory relay scenario
- **permission_esc** – injection attempts privilege escalation

### Key Findings

1. **Exposure is 100% – defenses are entirely downstream.** Across all 764 runs, every model exposed the token 100% of the time. No model prevented injection from entering its context.

2. **Qualitative differences between models.** Claude models (Haiku 4.5, Sonnet 4.5) stripped injections at `write_memory` summarisation (0/164 ASR). GPT‑4o‑mini propagated tokens with 53% ASR (95% CI: 41–65%). GPT‑5‑mini showed partial filtering but still 15% persistence.

3. **Same model, different attack surfaces – dramatically different results.** DeepSeek Chat scored 0/24 (0% ASR) on `memory_poison` but 8/8 (100% ASR) on `tool_poison` and `propagation`. A single attack‑surface evaluation could label DeepSeek as either “absolutely safe” or “completely vulnerable”—both misleading.

4. **All tested active defenses failed due to threat‑model mismatch.** Four conditions—`write_filter`, `pi_detector`, `spotlighting`, and combinations—all yielded 100% ASR. The issue is not design quality but that defenses target different injection surfaces than the actual attacks (e.g., `spotlighting` assumes document content, but injections come via function return streams).

5. **Claude relay nodes can “sanitise” downstream agents.** When a Claude model was placed as summariser/relay, 0 out of 40 propagation runs reached Agent B. This suggests that relay‑node security is composable—a secure relay can de‑contaminate any downstream agent regardless of its own safety level.

6. **“Goal drift” is a post‑hoc forensic signal, not a preventive one.** TF‑IDF cosine distance showed drift co‑occurs with harmful behaviour, not before it. A gradient‑boosting classifier achieved AUC=0.853 in‑distribution but collapsed to 0.39–0.57 in held‑out cross‑validation—essentially random guessing.

### Practical Deployment Potential

- The `agent_bench` framework and canary method can be directly adopted by enterprise security teams as a standard evaluation tool for multi‑agent systems. The authors plan to open‑source all benchmark code and logs.
- **“Relay de‑contamination rate”** is proposed as a first‑class metric for multi‑agent safety assessment.
- The **“surface coverage”** problem highlights evaluation blind spots—testing only a subset of attack surfaces can produce dangerously misleading safety conclusions.

---

## Technical Details

### Canary Token Tracking Mechanism

The `PropagationLogger` detects token presence using regex matching (`SECRET-[A-F0-9]{8}`) in every tool result and tool argument, emitting `canary_in_result` and `canary_in_args` events with stage labels. Each attacked run is annotated with the *furthest kill‑chain stage* reached. The gap between `EXPOSED` and `PERSISTED` identifies filtering at the summarisation stage; the gap between `PERSISTED` and `EXECUTED` identifies refusal at the execution stage.

### Metrics

- **ASR**: proportion of attacks where the canary reaches `EXECUTED`.
- **Task success rate**: proportion of runs that complete the legitimate task.
- **Relay de‑contamination rate**: proportion of injections stripped at the relay stage.
- All proportions reported with Wilson 95% confidence intervals.

### Models and Configuration

Five frontier models tested:
- GPT‑4o‑mini (`gpt-4o-mini-2024-07-18`)
- GPT‑5‑mini
- DeepSeek Chat v3 (`deepseek-chat`)
- Claude Haiku 4.5
- Claude Sonnet 4.5

Temperature = 0.0, max tokens = 2048. 764 runs across 9 batches (March 10–27, 2026): 428 no‑defense attacks, 80 with defenses, 256 clean controls. Sample sizes per cell (n=8–36) are moderate, but key findings replicated across independent batches over 17 days.

---

## Experimental Setup

### System Architecture

`agent_bench` core components:
- **MemoryStore**: key‑value with `read_memory(key)` and `write_memory(key, value)`. Supports `write_filter` (keyword scanning before commit). `seed(key, payload)` pre‑injects canary without triggering filter (for `memory_poison`).
- **ToolRegistry**: permission‑gated scheduler. Tools: `get_webpage(url)`, `query_db(sql)`, `send_report(to, body)`, `escalate_privilege(reason)`. Each has minimum privilege levels (READ/WRITE/EXECUTE/ADMIN).
- **MultiAgentOrchestrator**: coordinates Agent A and Agent B with two relay modes—delegation (A’s final message becomes B’s system prompt) and memory (A writes, B reads).
- **PropagationLogger**: JSONL event logger.

### Attack Scenarios

Four scenarios covering different injection surfaces:
- `memory_poison`: pre‑load via `MemoryStore.seed()`, task to summarise research notes and send email.
- `tool_poison`: injection in `get_webpage()` or `query_db()` returns.
- `propagation`: two‑agent memory relay.
- `permission_esc`: injection calls `escalate_privilege()` then issues high‑privilege operation.

---

## Comprehensive Analysis

This paper redefines prompt injection from a *model capability* problem to a *pipeline architecture* problem. Instead of asking “which model is safer?”, we should ask “at which stage of the pipeline does the safety mechanism engage?”.

**Methodologically**, the kill‑chain canary is elegant—it requires only a trackable token and regex matching, enabling stage‑level tracing at very low cost. Its simplicity aids adoption and standardisation.

**Empirically**, several findings stand out:

- The **100% exposure** fact implies that no model can prevent malicious input from entering its context. Security must be built into downstream stages rather than relying on input filtering.
- The **surface split** in DeepSeek reveals that model safety behaviour may be channel‑specific. Testing only one attack surface is dangerous.
- The **failure of active defenses** is structural: they defend against different attack surfaces than those actually used. Defences must be threat‑modelled against all incoming channels.
- **Goal drift** as a post‑hoc signal warns against relying on behavioural precursors for real‑time prevention.

**Architecturally**, the relay de‑contamination concept is highly actionable. Placing a secure model as a relay/summariser can protect the entire downstream pipeline, regardless of downstream agents’ safety levels.

---

## Practical Recommendations

1. **Multi‑agent security evaluation standards** – any deployment should include kill‑chain stage tracking, multi‑surface injection coverage, and relay de‑contamination as a first‑class metric.

2. **Design for “sanitising relays”** – use safer models at summarisation or relay nodes; this architectural choice protects downstream independently.

3. **Cover all attack surfaces in testing** – do not rely on a single injection channel; test every way external data enters the system.

4. **Do not over‑rely on proactive defence wrappers** – verify that their threat model covers all your injection vectors; surface coverage matters more than sophistication.

5. **Logging and forensics are the baseline** – since behavioural signals do not provide early warning, ensure detailed step‑level logs that trace tool‑call argument origins. The paper’s “source attribution heuristic” achieved 100% path reconstruction and zero false positives across 22 attacked runs.

---

## References

- Original paper: https://arxiv.org/abs/2603.28013
- PDF full text: https://arxiv.org/pdf/2603.28013v1
