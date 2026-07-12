# AgentAuditor: Human-level Safety and Security Evaluation for LLM Agents

## Paper Highlights

AgentAuditor, proposed by researchers from NYU Abu Dhabi and others at NeurIPS 2025, is a general, training‑free, memory‑augmented reasoning framework that enables LLM evaluators to reach human‑expert performance in safety and security assessment. It constructs “experience memories” and dynamically retrieves the most relevant reasoning traces to guide evaluation, while delivering state‑of‑the‑art results on its companion benchmark, ASSEBench.

---

## Core Research Content

### Problem Definition

LLM agents are evolving from pure text generators into autonomous systems that use tools, interact with environments, and make complex decisions. This evolution introduces significant challenges for safety and security evaluation, which existing methods face as four core dilemmas:

1. **Sequential risk accumulation** – individually safe steps can combine into unsafe outcomes.
2. **Semantic nuances** – implicit meanings in agent–environment interactions are often overlooked.
3. **Compound effects** – small issues can aggregate into major risks.
4. **Ambiguous safety rules** – evaluators struggle with context‑dependent and unclear safety criteria.

Rule‑based evaluators are efficient but inflexible, while LLM‑based evaluators suffer from inconsistency, bias, and limited interpretability. The authors characterise this as an **“evaluation crisis”** – agent complexity has outpaced our ability to reliably assess their safety and security.

### Innovative Method

The core hypothesis of AgentAuditor is that equipping an LLM evaluator with memory and reasoning capabilities, simulating the human expert’s assessment process, can achieve more reliable agent evaluation. The framework is training‑free and operates in three stages:

**Stage 1: Feature Memory Construction**  
For each agent interaction record, an LLM adaptively extracts three key semantic features – *Scenario*, *Risk Type*, and *Behaviour Pattern*. These features, together with the raw content, are then vectorised using models like Nomic‑Embed‑Text‑v1.5, forming a dual‑mode feature memory (FM) that combines structured semantics with vector embeddings for both interpretability and computational efficiency.

**Stage 2: Reasoning Memory Construction**  
The feature memory vectors undergo PCA dimensionality reduction and are clustered using the FINCH unsupervised algorithm to select representative samples (≈10% of the dataset). For each selected sample, high‑quality Chain‑of‑Thought (CoT) reasoning traces are generated – a prompt template guides the LLM to explain *why* the sample is safe or unsafe – forming a reasoning memory (RM) of “sample + label + CoT” that mimics the accumulated experience of human experts.

**Stage 3: Memory‑Augmented Reasoning**  
For a new interaction case, a two‑stage retrieval is performed: (1) top‑N initial filtering based on content embeddings to ensure semantic relevance, and (2) re‑ranking based on feature similarity (scenario, risk, behaviour) to refine the match. The top‑K CoT examples are retrieved to build a few‑shot prompt, guiding the LLM evaluator to produce accurate judgements.

**Companion Benchmark: ASSEBench**  
The team also constructed ASSEBench (Agent Safety & Security Evaluator Benchmark) – the first large‑scale benchmark covering both safety and security. It contains 2,293 carefully annotated interaction records, spanning 15 risk types, 528 interaction environments, 29 application scenarios, and 26 agent behaviour patterns. A distinctive feature is the adoption of both “Strict” and “Lenient” standards for ambiguous risk cases.

### Research Achievements

Experimental results show that AgentAuditor consistently improves LLM evaluation performance across all benchmarks and sets a new state‑of‑the‑art in LLM‑as‑a‑judge agent safety and security assessment. Its core achievement is **attaining human‑level evaluation accuracy**.

Specifically, AgentAuditor significantly boosts the capabilities of LLM evaluators through adaptive representative sample selection, structured memory, RAG‑augmented retrieval, and auto‑generated CoT. In various multi‑agent settings, AgentAuditor achieves up to **5% absolute accuracy improvement** over majority voting methods, and up to **3% improvement** over direct LLM‑as‑Judge baselines.

### Potential for Practical Deployment

The “training‑free” nature makes AgentAuditor highly practical – no expensive fine‑tuning is required; only a small set of annotated samples is needed to build the memory base. Using the same LLM throughout the pipeline ensures consistency. This design allows AgentAuditor to be:

- **Plug‑and‑play**: directly integrated into existing LLM evaluation workflows without modifying the underlying model.
- **Cost‑effective**: memories can be continuously enriched with new evaluation cases to improve performance over time.
- **Interpretable**: structured CoT reasoning traces provide clear justifications, alleviating the “black‑box” concern of LLM evaluators.

The project is open‑sourced at https://github.com/Astarojth/AgentAuditor-ASSEBench.

---

## Technical Details

### Framework Workflow
