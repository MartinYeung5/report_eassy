
# Unsafer in Many Turns: Benchmarking and Defending Multi‑Turn Safety Risks in Tool‑Using Agents

---

## 🧠 What This Paper Does

**LLM‑based agents** that invoke real‑world tools (filesystem, terminal, databases, browsers) become **significantly less safe** in multi‑turn interactions. This paper:

- Introduces **MT‑AgentRisk** – the first benchmark that evaluates safety in **multi‑turn + tool‑use** scenarios.
- Proposes **ToolShield** – a training‑free, tool‑agnostic defense that reduces multi‑turn attack success rates by **30%** on average.
- Reveals that multi‑turn attacks raise average ASR by **+16.1%** across six state‑of‑the‑art models, and that **stronger capability does not guarantee better safety** (DeepSeek‑v3.2 hits 85.4% ASR).

---

## 🔬 Key Contributions

### 1. Multi‑Turn Attack Taxonomy (MTA)
A systematic framework to transform single‑turn harmful tasks into multi‑turn attack sequences along three axes:
- **Format**: Addition / Decomposition
- **Method**: Mapping / Wrapping / Composition
- **Target**: Data / Files / Environment States

This yields **8 attack sub‑categories** that cover the vast majority of multi‑turn harmful intent patterns.

### 2. MT‑AgentRisk Benchmark
- **365 tasks** derived from OpenAgentSafety, SafeArena, P2SQL, MCPMark, etc.
- **5 tool environments**: Playwright‑MCP, Terminal, Filesystem‑MCP, PostgreSQL‑MCP, Notion‑MCP
- Average **3.19 turns** (2–7 turns), 71% require 3–4 turns.

### 3. ToolShield Defense
A self‑exploratory mechanism that lets the agent **use its own tool‑calling ability to detect misuse**:
1. **Synthesize** test cases via structured risk reasoning.
2. **Execute** them in a sandbox and observe effects.
3. **Distill** safety experiences from execution traces.
4. **Deploy** the experiences into the agent’s system context (e.g., `CLAUDE.md`).

---

## 📊 Key Results

| Metric | Value |
|--------|-------|
| Average ASR increase (multi‑turn vs. single‑turn) | **+16.1%** |
| Largest single‑model increase (Claude‑4.5‑Sonnet) | **+27.1%** |
| Highest multi‑turn ASR (DeepSeek‑v3.2) | **85.4%** |
| ToolShield average ASR reduction | **−30%** |
| ToolShield reduction on Claude‑4.5‑Sonnet | 72% → 22% (**−50%**) |
| False positive rate of ToolShield | **0%** (no impact on normal function) |

---

## 🛠️ Quick Start with ToolShield

ToolShield is available as a Python package – no API keys, no sandbox setup, no fine‑tuning required.

```bash
pip install toolshield
```

### Import pre‑generated safety experiences for an existing agent
```bash
toolshield import --exp-file filesystem-mcp.json --agent claude_code
```

### Generate experiences for a new tool (self‑exploration mode)
```bash
export TOOLSHIELD_MODEL_NAME="anthropic/claude-sonnet-4.5"
export OPENROUTER_API_KEY="your-key"
toolshield --mcp_name postgres --mcp_server http://localhost:9091 \
          --output_path output/postgres --agent codex
```

### Unload experiences
```bash
toolshield unload --agent claude_code
```

**Supported agents**: Claude Code, Codex, OpenClaw, Cursor, OpenHands  
**Pre‑generated for**: 6 models (Claude‑4.5‑Sonnet, GPT‑5.2, DeepSeek‑v3.2, Gemini‑3‑Flash, Qwen3‑Coder‑Plus, Seed‑1.6) and the 5 tools above.

---

## 🔍 Why This Matters

### The Multi‑Turn + Tool Attack Surface
Traditional safety filters check *what the user says*. In tool‑using agents, we must check *what the agent does* – tool calls, parameters, file changes, state mutations. A single turn may be benign, but a sequence of turns can together execute a forbidden action (e.g., `rm -rf /` decomposed into three separate function definitions and a final call).

### The Capability‑Safety Paradox
The paper’s most striking finding: **models with stronger reasoning and tool‑handling capabilities are often more vulnerable** to multi‑turn attacks. DeepSeek‑v3.2 – the top‑performing open‑source model on benchmarks – suffers the highest ASR (85.4%). This challenges the common assumption that “stronger = safer”.

### ToolShield’s Design Philosophy
Instead of adding external guardrails that don’t understand the agent’s context, ToolShield **reuses the agent’s own intelligence** to explore and remember safe/unsafe tool‑use patterns. This approach is:
- **Training‑free** – no expensive fine‑tuning.
- **Tool‑agnostic** – works with any MCP‑compatible tool.
- **Adaptive** – automatically generates safety knowledge for new tools.

---

## 📦 Practical Deployment Tips

1. **Always test multi‑turn safety before production** – single‑turn evaluations are insufficient.
2. **Monitor execution traces** – not just inputs/outputs, but tool‑call sequences and state changes.
3. **Make ToolShield a standard layer** – its zero‑false‑positive and zero‑cost nature makes it an ideal default for any tool‑based agent.
4. **Choose models wisely** – don’t rely solely on capability scores; run multi‑turn safety benchmarks (like MT‑AgentRisk) as part of your selection process.

---

## 🔗 References

- Original paper: [arXiv:2602.13379](https://arxiv.org/abs/2602.13379)
- Project page: [unsafer-in-many-turns.github.io](https://unsafer-in-many-turns.github.io/)
- Official code: [github.com/CHATS-lab/ToolShield](https://github.com/CHATS-lab/ToolShield)
- ICML 2026 page: [icml.cc/virtual/2026/poster/62233](https://icml.cc/virtual/2026/poster/62233)
- Hugging Face paper page: [huggingface.co/papers/2602.13379](https://huggingface.co/papers/2602.13379)
```
