
# Watermark Stealing in Large Language Models – Analysis

This document provides a comprehensive, expert‑level summary and analysis of the paper:

> **Watermark Stealing in Large Language Models**  
> Nikola Jovanović, Robin Staab, Martin Vechev (ETH Zurich SRI Lab)  
> ICML 2024 · [Paper](https://arxiv.org/abs/2402.19361) · [Project](https://watermark-stealing.org/) · [Code](https://github.com/eth-sri/watermark-stealing)

---

## 📄 Paper Synopsis

The authors introduce **Watermark Stealing (WS)** – the first systematic attack that reverse‑engineers the watermarking rule of a black‑box LLM through API queries, at a cost of less than $50. Once stolen, the approximate watermark can be used to launch both **spoofing** (forge watermarked text) and **scrubbing** (remove watermarks) attacks with >80% success. This challenges the prevailing assumption that current watermark schemes are ready for real‑world deployment.

---

## 🧠 Core Contributions

### Problem Definition

LLM watermarks are widely considered a practical solution for detecting AI‑generated content and tracing provenance. Major tech companies and regulators (e.g., US Executive Order, EU AI Act) are pushing for their adoption. However, this paper shows that **the robustness of existing schemes against determined adversaries has been overestimated**.

The critical threat: an attacker who can repeatedly query a watermarked LLM can statistically infer the hidden green‑/red‑list partitioning rule – i.e., “steal” the watermark. Once stolen, the attacker can both fake watermarks and strip them from genuine watermarked texts. No prior work had systematically automated this theft or demonstrated that theft also dramatically improves scrubbing.

### Innovative Approach

The paper proposes the first **automatic watermark‑stealing algorithm**, used in a two‑step attack pipeline:

**(1) Watermark Stealing (Step 1)**

The attacker collects API responses from the victim model \(LM_{mo}\) using a set of crafted prompts. From these responses, they build an approximate scoring function \(s^*(T, [T_1 T_2 \dots T_h])\) that estimates, for a given context \(h\)-gram, how likely a candidate token \(T\) belongs to the green list.

**(2) Downstream Attacks (Step 2)**

Armed with \(s^*\), the attacker combines it with a public auxiliary model \(LM_{att}\) (no more queries to the victim are needed):

- **Spoofing Attack**: During generation, bias the logits of candidate tokens according to \(s^*\) – higher scores (more likely green) receive a boost:
  \[
  l'_T = l_T + \delta_{\text{att}} \cdot s^*(T, [T_1 \dots T_h])
  \]
  This lets the attacker produce high‑quality text on arbitrary topics that will be classified as watermarked by the detector.

- **Scrubbing Attack**: Using knowledge of \(s^*\), the attacker rewrites watermarked text into a non‑watermarked version while preserving semantics. Previous work thought scrubbing long texts was difficult; the stolen watermark improves success from ~0% to >80%.

### Key Results

The paper evaluates against state‑of‑the‑art schemes: KGW‑SOFT, KGW‑HARD, KGW2‑SUM, KGW2‑SELFHASH, and UNIGRAM. Notable findings:

- **Cost**: The one‑time stealing cost is **under $50** at ChatGPT‑style pricing.
- **Spoofing success** >80% even under the strictest threat model (no detector access, no baseline responses) against the previously recommended KGW2‑SELFHASH.
- **Scrubbing success** >80% on long texts, while all baselines stay below 25%.
- **Breaks the trade‑off hypothesis**: Earlier work suggested a trade‑off between spoofing and scrubbing robustness. This paper shows that once the watermark is stealable, both attacks can be simultaneously achieved at high rates.

### Practical Implications

- **For providers**: The paper is a wake‑up call – any planned watermark deployment must treat “watermark stealing” as a primary threat.
- **For researchers**: The public code and benchmark provide a baseline for designing more robust future watermarks.
- **Limitation**: The authors acknowledge that no known production‑scale LLM watermark system exists today; however, this only highlights the urgency to harden schemes **before** they are widely deployed.

---

## ⚙️ Technical Details

### Watermark Background (KGW‑type)

Most current schemes modify the output distribution:

- At each generation step, a pseudo‑random function (PRF) \(f\) splits the vocabulary \(V\) into a **green list** (\(\gamma |V|\) tokens) and a **red list** (the rest).
- The PRF seed is derived from the hash of the previous \(h\) tokens combined with a secret key \(\xi\).
- Generation is biased toward green tokens (e.g., logit boost \(\delta\) in KGW‑SOFT, or forced selection in KGW‑HARD).
- Detection computes the z‑statistic:
  \[
  z = \frac{n_{\text{green}} - \gamma L}{\sqrt{L\gamma(1-\gamma)}}
  \]
  where \(L\) is text length; a p‑value below a threshold indicates watermarked text.

KGW2‑SELFHASH (recommended by Kirchenbauer et al. 2024) uses:
\[
\min\{H(T_{t-h}), \dots, H(T_t)\} \cdot \xi \cdot H(T_t)
\]
as the PRF seed.

### Stealing Algorithm Intuition

The attacker cannot know \(\xi\), but they can estimate the green‑list probability for each token in a given context by collecting many responses and measuring token frequency anomalies. The scoring function \(s^*\) is built from these statistical cues. Then, during spoofing, the attacker injects \(s^*\) into the auxiliary model’s decoding – effectively mimicking the victim’s bias.

### Threat Model Dimensions

| Dimension | Variants | Description |
|-----------|----------|-------------|
| **Detector access** | D0 (none) <br> D1 (API) | Detector is private; attacker can or cannot query it for verification. |
| **Baseline responses** | B0 (unavailable) <br> B1 (available) | Attacker has (or lacks) access to pre‑watermark responses from the same model. |

The paper focuses on the **most stringent setting (D0, B0)** – no detector access, no baseline.

---

## 🧪 Experimental Setup

- **Models**: Open‑source LLMs from 1.3B to 30B parameters, including instruction‑tuned variants.
- **Cost estimation**: Based on January 2024 ChatGPT pricing.
- **Attack budget**: < $50 total query cost.
- **Public code**: All experiments are reproducible via the [GitHub repo](https://github.com/eth-sri/watermark-stealing).

Key evaluation aspects:
- Diverse prompt topics (harder than arbitrary text generation).
- Multi‑key scenarios (victim switches among \(k\) keys per response).
- Paraphrasing tasks (inject watermark into non‑watermarked text while paraphrasing).
- Harmful content generation (even safety‑aligned models can be tricked into producing watermarked toxic text).

---

## 🔍 Comprehensive Analysis

### Academic Significance

This work is a landmark in LLM watermark security. It:
- **Unifies** spoofing and scrubbing under a single threat framework – stealing.
- **Refutes** the trade‑off hypothesis – both attacks become powerful once the watermark is stolen.
- **Highlights** the insufficiency of Kerckhoffs’ principle: even with a secret key, statistical inference can leak enough information to break the system.

### Practical Takeaways

- **Secrecy of the algorithm is not enough** – attackers can reverse‑engineer it from queries.
- **Future designs** must consider dynamic keys, multi‑key schemes, or more complex dependencies to raise the bar for theft.
- **Detection‑side defences** (e.g., anomaly detection on queries) could complement algorithmic improvements.

### Limitations and Future Work

The authors are transparent about limitations:
- No known deployment of these schemes in production – but this is the best time to fix flaws.
- Theft assumes consistent watermark parameters – adaptive or changing rules may increase difficulty.
- Future directions: detection of spoofed watermarks, stronger multi‑key strategies, and adaptive stealing.

---

## 🛠️ Practical Applications

### For Model Providers

1. **Include watermark stealing in red‑team testing** – simulate the attack before deployment.
2. **Rate‑limit API queries** – high‑volume repeated prompts are a tell‑tale sign.
3. **Consider periodic key rotation** – increase the cost and time for attackers.
4. **Monitor query patterns** – unusual clusters of similar prompts may indicate stealing attempts.

### For Researchers

- **Benchmark new watermarks against this attack** – use the provided code.
- **Explore theft‑resistant designs** – e.g., schemes that remain robust even if the green list is partly leaked.
- **Develop detection for forged watermarks** – distinguish genuine from fake.

### Resources

- Official project: [https://watermark-stealing.org](https://watermark-stealing.org)
- Code: [https://github.com/eth-sri/watermark-stealing](https://github.com/eth-sri/watermark-stealing)
- Replication package: [https://github.com/juanbelieni/watermark-stealing-replication](https://github.com/juanbelieni/watermark-stealing-replication)

---

## 📚 References

- Jovanović, N., Staab, R., & Vechev, M. (2024). Watermark Stealing in Large Language Models. *Proceedings of the 41st International Conference on Machine Learning*, PMLR 235:22570‑22593.
- arXiv preprint: [https://arxiv.org/abs/2402.19361](https://arxiv.org/abs/2402.19361)
- ICML 2024 poster: [https://icml.cc/virtual/2024/poster/33848](https://icml.cc/virtual/2024/poster/33848)


*This summary is intended for researchers and practitioners seeking a concise yet thorough understanding of the paper’s contributions, methodology, and implications.*
```
