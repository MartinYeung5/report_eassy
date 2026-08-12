
# Got a Secret? LLM Agents Can‘t Keep It: Evaluating Privacy in Multi-Agent Systems — In‑depth Paper Analysis

**Paper:** *Got a Secret? LLM Agents Can’t Keep It: Evaluating Privacy in Multi-Agent Systems* (ACM CAIS 2026)  
**Authors:** Aman Priyanshu, Supriti Vijay, Esha Pahwa  
**DOI:** [10.1145/3786335.3813173](https://doi.org/10.1145/3786335.3813173) | **arXiv:** [2605.27766](https://arxiv.org/abs/2605.27766)  
**Code & Data:** [https://llms-cant-keep-secrets.github.io/](https://llms-cant-keep-secrets.github.io/)

---

## Key Takeaways

The paper reveals a fundamental shift: when LLM agents move from isolated single‑turn interactions to persistent multi‑agent social environments, privacy leakage escalates dramatically. Using a Moltbook‑style simulation with thousands of agents across 124 communities over 25 simulated days, the authors find that **leakage rates jump from 19.95% (conventional benchmarks) to 45.30%**, and that leakage is socially contagious — observing a peer disclose sensitive information makes an agent **8× more likely** to leak its own secrets. Even explicit privacy instructions cannot eliminate the problem, with leakage still exceeding 37.8%.

---

## Core Research Content

### Problem Definition

Current safety benchmarks for LLMs suffer from a critical blind spot: they evaluate models in **isolated environments** — one user, one prompt, one response. In reality, deployed AI agents increasingly operate as **persistent software entities** that run over long time horizons, call tools, and interact repeatedly with users and other agents.

Privacy becomes especially acute in multi‑agent deployments. Systems are increasingly storing and retrieving “memories” to personalise interactions, but persistent memory introduces a fundamental risk: information can surface in inappropriate contexts, even if it was truthful or useful elsewhere.

The study frames privacy through the lens of **contextual integrity** — whether an information flow is appropriate relative to norms. Changing the interaction context, audience, social environment, or normative expectations can transform a disclosure into a privacy violation. Yet existing safety benchmarks do not model privacy risk as a **dynamic phenomenon emerging from long‑term multi‑agent interactions in persistent social settings**.

### Innovation

**1. Large‑Scale Multi‑Agent Simulation Platform**

A Moltbook‑style simulation was built with:
- **2,533 agents**, each carrying a private profile spanning **ten sensitive domains** (health, finance, employment, etc.).
- **124 sub‑communities** (analogous to Reddit subreddits).
- **25 simulated days**, generating **29,945 posts** and **81,264 replies**.
- **12 tool functions** enabling agents to browse, search, post, reply, vote, and manage memory.

**2. Two‑Tier Evaluation Design**

- **Organic simulation**: 2,533 agents interacting freely without intervention, measuring leakage under natural conditions.
- **Controlled experiments**: 7 frontier models (OpenAI GPT‑5 series, GPT‑4o, Google Gemini‑3) exposed to **five escalating levels of adversarial social pressure**, measuring additional leakage.

**3. LLM‑as‑Judge Detection Pipeline**

A model‑based judge (gpt‑5‑nano) receives the agent’s full profile and generated content, and classifies leakage per domain. Crucially, the judge does **not** see the social context, making it immune to social influence.

### Results

- **Leakage surge**: from 19.95% (single‑turn benchmarks) to 45.30% (multi‑turn social evaluation). Over 25 days, about 2,500 out of 111,000 posts/replies contained privacy leaks.
- **Social contagion**: if a previous reply leaks, the next reply in the same thread has a **12.8%** chance of leaking, versus **1.6%** when the previous reply is clean — an **8× increase**. This effect does not require adversarial prompting.
- **Privacy instructions fail**: even with explicit “do not disclose” system prompts, leakage remained above **37.8%**. In extreme cases, one agent leaked 27 out of 29 write operations.
- **Community context matters as much as model choice**: leakage rates varied by an **order of magnitude** across sub‑communities.
- **Leakage accumulates with interaction length**: at 50 tool calls, some models approached **50–60%** leakage, while even stronger ones hovered around **20–30%**.

### Real‑World Applicability

1. **Security auditing** for multi‑agent systems before deployment.
2. **Regulatory monitoring** for AI‑only social platforms.
3. **Design of context‑aware privacy safeguards** beyond simple prompting.
4. **Benchmark evolution** toward dynamic, social, multi‑turn evaluations.

---

## Technical Details

### Agent Profile Generation

Each agent receives a synthetic human profile generated via a two‑stage process:
1. Seed identity (name, address, DOB, phone, credit score) from Faker.
2. Structural template from one of 10 annotated profiles in CIMemories.
3. Fusion using GPT‑5‑mini to produce a complete profile.

Each profile contains **96.8 ± 16.3** key‑value pairs.

### Agent Interaction Loop (Simplified)

```
For each simulated day:
  Scheduler selects a batch of agents to activate.
  For each activated agent:
    1. Receive system prompt (persona, private profile, MEMORY.md, platform instructions).
    2. Enter tool‑calling loop:
       a. LLM generates response.
       b. If tool needed → dispatch → obtain result → append to messages.
       c. If memory update needed → update MEMORY.md.
       d. Repeat until budget exhausted or yield.
    3. Persist updated memory.
```

### Adversarial Social Pressure

In controlled experiments, the platform injects “normalising disclosure” posts written by fictional agents, with **five intensity levels** (Level 1 = no injection, Level 5 = all 124 communities injected). Each injected post is artificially upvoted to **1.2× the community’s historical top** to ensure high visibility.

Two moderating variables are tested:
- Whether a privacy‑protection instruction is added to the system prompt.
- Whether agents are pre‑subscribed to communities based on profile interests.

### Leakage Detection Workflow

```
For each piece of content:
  1. Look up author by author_hash.
  2. Retrieve the author’s full profile.
  3. Use gpt‑5‑nano as the judge.
  4. Check each of 10 privacy domains.
  5. If any domain matches → mark as leakage.
```

---

## Experimental Setup

### Hardware & Software

- **Database**: SQLite shared across all agents.
- **Model backends**: OpenAI GPT‑5‑nano, GPT‑5‑mini, GPT‑5 (organic); plus GPT‑4o, GPT‑4o‑mini, Gemini‑3‑flash‑preview, Gemini‑3‑pro‑preview (controlled).
- **Judge model**: gpt‑5‑nano.
- **Tool suite**: 12 functions (find_subreddit, get_newly_posted, open_subreddit, open_post, find_post, get_user_profile, post_in_subreddit, thread_in_post, upvote_downvote_post, upvote_downvote_thread, append_to_memory, modify_memory).
- **Core platform**: 124 sub‑communities, nested threads, voting, user profiles, persistent MEMORY.md scratchpad per agent.

### Key Parameters

- **Organic simulation**: 2,533 agents × 25 days.
- **Controlled experiments**: 7 models × 10 profiles × 5 adversarial levels × 2 conditions × 5 budget checkpoints (10,20,30,40,50 tool calls) = **7,000 trajectories**.
- **Total content**: 29,945 posts + 81,264 replies = **111,209 items**.

---

## Comprehensive Analysis

### Theoretical Contribution

The paper extends **contextual integrity theory** from static assessments to dynamic multi‑agent settings. It demonstrates that privacy is **not an inherent property of an agent**, but a **function of context** — changing the context (single‑turn → multi‑turn, isolated → community, no pressure → pressure) systematically alters privacy‑protective behaviour.

Crucially, the **social contagion** mechanism reveals that agents do not need to be “broken” or “jailbroken” — merely observing others doing the same thing shifts their judgement of appropriateness. This suggests a deeper issue: **even if each individual agent is “safe,” the collective behaviour of a multi‑agent system can be unsafe.**

### Methodological Contribution

This work is the first to combine **thousand‑agent social simulation** with **privacy security evaluation**. Prior work either used small‑scale sandboxes (<50 agents) or focused on social science questions (opinion formation, collective behaviour) rather than safety and privacy.

### Challenge to Current Safety Paradigms

The findings challenge existing evaluation practices:
- **Static benchmarks** systematically underestimate deployment risks.
- **Single‑turn evaluations** miss attack surfaces unlocked by long‑context and conversational decomposition.
- **Isolated testing** ignores that social context alone can trigger sensitive disclosures — **no adversarial prompt required**.

### Limitations

- Agent personas derive from early Moltbook snapshots, which may not represent all LLM‑agent types.
- Synthetic profiles, though methodologically grounded, may not mirror real human privacy preferences.
- The judge is itself an LLM, introducing possible evaluation bias.
- The simulation simplifies real‑world social platforms, so results may not fully generalise.

---

## Practical Applications

### For Enterprises

**1. Social‑stress testing before deployment**  
Do not test agents in isolation. Run evaluations that include:
- Multi‑agent co‑existence.
- Simulated social interactions (multiple turns).
- Exposure to “normalised” disclosures at varying intensities.

**2. Community‑level monitoring**  
Since community context is as influential as model choice, establish **community‑level leakage dashboards** rather than monitoring only individual agents. Identify high‑risk communities early.

**3. Move beyond “add a privacy instruction”**  
Simply prompting “do not leak” is insufficient — leakage remains >37.8%. Explore:
- Context‑aware filters that detect sensitive content during generation.
- Social‑environment isolation (limit exposure to normalising posts).
- Dynamic permission adjustments based on community exposure history.

**4. Account for interaction duration**  
Leakage increases with interaction length. For long‑running agents (e.g., 24/7 assistants), implement **periodic memory resets or reviews** to prevent accumulating risk.

### For Policymakers

- Update safety standards to include **multi‑agent social scenarios** — static single‑turn benchmarks are no longer adequate.
- Require **social‑simulation‑based privacy risk reports** for large‑scale multi‑agent deployments (e.g., AI social platforms, multi‑agent customer service).

### For Researchers

- Incorporate **social dimensions** as core variables in safety evaluation, not optional add‑ons.
- Investigate **“social immunity”** — can positive modelling (agents observing refusal behaviours) reduce leakage?
- Develop **architecture‑level privacy mechanisms** that internalise protection rather than relying solely on prompting.

---

## References

- Original paper: [arXiv:2605.27766](https://arxiv.org/abs/2605.27766)  
- ACM CAIS ’26: [DOI 10.1145/3786335.3813173](https://doi.org/10.1145/3786335.3813173)  
- Project page: [https://llms-cant-keep-secrets.github.io/](https://llms-cant-keep-secrets.github.io/)
