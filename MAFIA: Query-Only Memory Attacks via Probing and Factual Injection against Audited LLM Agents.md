
# MAFIA: Query-Only Memory Attacks via Probing and Factual Injection against Audited LLM Agents

**A comprehensive analysis of the paper:** [MAFIA: Query-Only Memory Attacks via Probing and Factual Injection against Audited LLM Agents](https://arxiv.org/abs/2608.03844) (2026)

---

## Key Points

MAFIA introduces a query‑only memory attack framework against LLM agents equipped with external memory modules. It uses **retrieval probing** to locate optimal injection positions and **compact factual cloaks** to bypass input auditing. Under realistic constraints – a large benign memory pool and a strong auditor – MAFIA achieves up to 90.7% attack success rate while reducing the auditor’s detection rate from 83.3% to below 7.4%.

---

## Core Research Contents

### Problem Definition
Memory‑augmented LLM agents rely on external memory to store historical interactions. Existing query‑based attacks (e.g., MINJA) fail under two realistic conditions: (1) benign records overwhelm the top‑K retrieval slots, and (2) input auditors detect and block explicit instructions or adversarial signals. MAFIA tackles both **retrieval competitiveness** and **audit evasion**.

### Innovation Highlights
A two‑stage attack framework:

- **Placement via Retrieval Probing** – Black‑box probing infers the semantic distribution of the memory space. Hierarchical clustering identifies dense semantic regions, and a size‑ranked round‑robin allocation distributes the limited injection budget to regions most likely to be retrieved by future victim queries.
- **Compact Factual Cloak Payload** – Uses the template `patient V (→patient T in this DB)` to disguise malicious mappings as database facts rather than instructions. This preserves semantic proximity to victim queries while evading LLM‑based auditors.

### Key Results
Tested on four agent‑dataset configurations (EHRAgent on MIMIC‑III/eICU, RAP on WebShop, DataInterpreter on HuggingFace Hub):

- Maintains **69.63%–95.56%** injection success rate and **56.66%–90.74%** attack success rate even after auditing.
- Per‑record detection rate drops from **68.40%–83.30%** (MINJA) to **0.00%–7.40%**.
- Effective across different backbone LLMs (GPT‑5.4‑mini, Claude‑Sonnet‑4.6, Gemini‑3.1‑pro, Kimi‑K2.6) and retrieval encoders.
- RIR@4 significantly outperforms MINJA on all tested retrievers.

### Real‑World Applicability
- **For defenders** – Provides a concrete attack model to design more robust input auditing and memory‑side defences.
- **For red‑team testing** – Offers a reproducible penetration testing methodology for RAG‑based systems.
- The code is open‑sourced at [https://github.com/JiamingChen1234/MAFIA](https://github.com/JiamingChen1234/MAFIA).

---

## Technical Details

### 1. Memory Probing
The attacker only has query access. They synthesise `K_p` diverse seed queries and extract retrieved historical questions from the agent’s responses using a black‑box extraction method (adapted from MEXTRA). The collected surface‑question set **D** is used for subsequent clustering.

### 2. Clustering & Budget Allocation
Embedding vectors from **D** are grouped via **agglomerative clustering**. Cluster sizes estimate the representational strength of each region. A **size‑ranked round‑robin** allocation then selects injection templates: clusters are sorted by size (descending), and templates are chosen by picking the unselected question closest to the cluster centre, alternating across clusters until the budget `b` is exhausted.

### 3. Injection Scheduling
Templates are injected in **ascending order of cluster size** (small clusters first). Sparse regions (weaker benign competition) allow V→T records to be retrieved earlier, serving as demonstrations that reinforce goal‑directed generation during later large‑cluster attacks. Experiments show ascending order yields 75.40% ASR vs. 60.19% for descending.

### 4. Compact Factual Cloak
Payload template: `patient V (→patient T in this DB)`.

- **Parentheses** – Frame the cloaking as supplementary schema annotations, seen as benign by auditors.
- **Arrow** – Encodes the replacement relation without action verbs that trigger audit alarms.
- **“in this DB”** – Elevates the mapping to a local fact, so the agent adopts it as task evidence.

Because the three components are compactly inserted, the base query remains dominant in the embedding space, keeping the cloaked record within the semantic neighbourhood of victim queries.

### 5. Key Metrics

| Metric | Description |
|--------|-------------|
| RIR@K | Retrieval hit rate of malicious records in top‑K results |
| ̄np@K | Average number of poisoned records in top‑K |
| ISR | Injection success rate (passed audit and written) |
| ASR | Attack success rate (victim query triggers targeted response) |
| DR | Detection rate (flagged by the defender) |

---

## Experimental Setup

### Agents & Datasets
Uses **gpt‑5.4‑mini** as backbone. Four configurations:

| Agent | Dataset | Attack Goal |
|-------|---------|-------------|
| EHRAgent | MIMIC‑III | Patient record redirection (SUBJECT_ID replacement) |
| EHRAgent | eICU | Drug replacement to dangerous prescriptions |
| RAP | WebShop | Shopping query redirection to target brand |
| DataInterpreter | HuggingFace Hub | Software‑engineering dependency replacement |

Each configuration instantiates 9 (victim, target) pairs.

### Memory Pool & Retriever
- **MIMIC‑III/eICU**: ~5.8K EHRSQL benign records.
- **WebShop**: 2,000 records.
- **DataInterpreter**: 2,000 records from instruction‑tuning and code benchmarks.
- **Retriever**: e5‑small‑v2 with cosine similarity, K=4 (K=3 for WebShop).

### Input Auditor
Each candidate record is screened by a **gpt‑5.4‑mini** auditor using an injection‑detection prompt; records classified as `INJECTION` are discarded.

### Attack Budgets
- **Probing**: 300 queries amortised over 9 pairs.
- **Injection**: 90 attacks per pair for MIMIC‑III, 30 per pair for others.

---

## In‑Depth Analysis

### Academic Contribution
MAFIA is the first to systematically evaluate query‑only memory attacks under the **dual constraints** of a large benign memory pool and a strong input auditor. Prior work (MINJA) fails in these realistic settings. MAFIA’s key insight is to **turn structural properties of memory – persistence and scale – into attack opportunities**: persistence keeps accepted poisoned records, while scale allows probing to reveal semantic distributions for targeted placement.

### Methodological Elegance
The two‑stage design strikes a fine engineering trade‑off. **Placement** solves *where* to inject via black‑box probing, highly practical under query‑only access. **Payload** solves *how* to inject by balancing semantic proximity (for retrieval) and audit evasion (by avoiding explicit instructions). The post‑audit ISR (69.63%–95.56%) confirms the payload’s robustness, whereas MINJA’s post‑ISR often drops to 0%.

### Defence Implications
The paper also evaluates the memory‑side defence A‑MemGuard, which detects 75.6% of MAFIA appearances but with a 44.9% false‑positive rate – too high for deployment. This reveals a fundamental dilemma: **over‑sensitive defences hurt user experience, while loose ones cannot stop MAFIA**. Achieving an acceptable balance remains open.

### Limitations & Future Work
MAFIA focuses on RAG‑based memory systems; its effectiveness on emerging long‑term memory (LTM) systems is not yet verified. Moreover, the tested auditors are LLM‑based text‑level detectors – whether MAFIA evades more sophisticated multi‑modal or behavioural auditors requires future exploration.

---

## Practical Recommendations

### For Security Researchers & Defenders
- **Strengthen auditing** – current LLM auditors are weak against “factual‑cloaked” attacks. Incorporate **semantic consistency checks** (e.g., verify if the query’s intent matches its surface form).
- **Memory‑side defence** – although A‑MemGuard has high false positives, its retrieval‑time consistency checking is promising. Consider a **dual‑layer** architecture combining write‑time auditing and retrieval‑time verification.
- **Budget monitoring** – since MAFIA uses a limited injection budget (30–90 per pair), **frequency‑based anomaly detection** can serve as a supplementary defence.

### For AI System Developers
- **Memory isolation** – isolate memories across user sessions to prevent cross‑session poisoning.
- **Hardened retrieval** – introduce **diversity penalties** or **source‑credibility weighting** in retrieval strategies.
- **Input normalisation** – pre‑process special patterns like parentheses and arrows to reduce the effectiveness of factual cloaks.

### For Red‑Team Testing
MAFIA provides a systematic methodology:

1. **Probe** – map the semantic distribution of the target memory.
2. **Place** – inject cloaked payloads based on clustering results.
3. **Schedule** – use small‑cluster‑first injection to reinforce downstream attacks.

This methodology is generalisable to security assessments of other RAG systems.

---

## References

- Original paper: Chen, J., Gao, Y., Li, Y., Liu, Z., Zhang, Y., & Zhang, J. (2026). MAFIA: Query-Only Memory Attacks via Probing and Factual Injection against Audited LLM Agents. *arXiv preprint arXiv:2608.03844*. [https://arxiv.org/abs/2608.03844](https://arxiv.org/abs/2608.03844)
- Code repository: [https://github.com/JiamingChen1234/MAFIA](https://github.com/JiamingChen1234/MAFIA)
