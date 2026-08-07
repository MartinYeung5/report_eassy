# JudgeDeceiver: Optimization-Based Prompt Injection Attack to LLM-as-a-Judge (ACM CCS 2024)

[![Paper](https://img.shields.io/badge/Paper-ArXiv-red)](https://arxiv.org/abs/2403.17710)
[![Code](https://img.shields.io/badge/Code-GitHub-blue)](https://github.com/ShiJiawenwen/JudgeDeceiver)

## TL;DR

LLM-as-a-Judge is widely adopted in search ranking, RLAIF, and tool selection. This paper presents **JudgeDeceiver** – an optimization-based prompt injection attack that automatically crafts injection sequences, forcing the judge LLM to select the attacker‑specified response regardless of other candidates. Experiments show an average attack success rate of **90.8%** across multiple LLMs and benchmarks, and existing defenses are largely ineffective.

---

## Core Contributions

### Problem Definition

The LLM-as-a-Judge pipeline takes a question \( q \) and a set of candidate responses \( R=\{r_1,\dots,r_n\} \), and asks the judge LLM to select the best one. However, this mechanism is vulnerable to manipulation: Can an attacker inject a carefully crafted sequence into one candidate response to systematically control the judge’s decision?

Manual prompt injection attacks (e.g., naive attack, escape characters, context ignoring) have limited success. The core question addressed by JudgeDeceiver is: **Can we formulate prompt injection as a solvable optimisation problem, enabling automated and high‑success‑rate attacks?**

### Innovation

**JudgeDeceiver transforms prompt injection into a differentiable optimisation problem.** Key innovations:

1. **Shadow candidates generation** – The attacker cannot access the real candidate set, so they use a publicly available LLM to generate a set of “shadow candidates” that simulate the real scenario.

2. **Triple‑component loss function** – The optimisation minimises a weighted sum of three losses:
   - **Target‑aligned generation loss** (\( \mathcal{L}_{aligned} \)): increases the probability that the judge outputs the attacker‑desired verdict.
   - **Target‑enhancement loss** (\( \mathcal{L}_{enhancement} \)): improves robustness against variations in the position of the target response.
   - **Adversarial perplexity loss** (\( \mathcal{L}_{perplexity} \)): reduces the perplexity of the injected sequence to evade perplexity‑based detection.

3. **Gradient‑based solver** – A greedy coordinate gradient (GCG‑like) algorithm iteratively replaces tokens based on gradient information. It also employs **position‑adaptive** and **stepwise optimisation** strategies to ensure success under various positional configurations.

### Results

- **Attack success rate**: On the MT‑Bench dataset with Mistral‑7B as the judge, the average attack success rate reaches **90.8%**, and the positional consistency is **83.4%**.
- **Comparison**: Significantly outperforms manual prompt injection attacks and conventional jailbreak methods adapted to this task.
- **Real‑world scenarios**: High success rates are achieved in three practical applications – LLM‑powered search, RLAIF, and tool selection.
- **Defence evaluation**: Known‑answer detection, perplexity filtering, and sliding‑window perplexity all fail to reliably detect JudgeDeceiver. For instance, with Llama‑3‑8B as the judge, sliding‑window perplexity (at <1% false positive rate) misses 70% of the target responses containing the injected sequence.

### Practical Impact

JudgeDeceiver exposes serious risks with concrete implications:

- **Model leaderboard manipulation** – Attackers can boost their own model’s ranking when evaluated by LLM‑as‑a‑Judge.
- **Search engine poisoning** – Attacker‑controlled web pages can be promoted in LLM‑powered search results.
- **RLAIF data poisoning** – Malicious data injected during RLHF can corrupt the alignment of the target model.
- **Tool selection hijacking** – Attackers can increase the invocation frequency of their own tools in LLM‑agent systems.

---

## Technical Details

### Formal Problem Setup

The judge LLM \( E \) processes the prompt:

\[
E(p_{\text{header}} \oplus q \oplus r_1 \oplus r_2 \oplus \cdots \oplus r_n \oplus p_{\text{trailer}}) = o_k
\]

where \( o_k \) indicates that the judge selects response \( r_k \).

The attacker aims to add an injection sequence \( \delta \) to the target response \( r_t \) so that the judge outputs the index \( t \):

\[
E(p_{\text{header}} \oplus q \oplus r_1 \oplus \cdots \oplus \mathcal{A}(r_t, \delta) \oplus \cdots \oplus r_n \oplus p_{\text{trailer}}) = o_t
\]

### Optimisation Objective

The total loss is:

\[
\mathcal{L}_{total}(\mathbf{x}^{(i)}, \delta) = \mathcal{L}_{aligned} + \alpha \mathcal{L}_{enhancement} + \beta \mathcal{L}_{perplexity}
\]

\[
\min_{\delta} \mathcal{L}_{total}(\delta) = \sum_{i=1}^{M} \mathcal{L}_{total}(\mathbf{x}^{(i)}, \delta)
\]

where \( M \) is the number of shadow candidate sets used for robust optimisation.

### Gradient Update

For the \( j \)-th token in \( \delta \), the gradient is computed as:

\[
\nabla_{T_j} \mathcal{L}_{total}(\delta) \in \mathbb{R}^{|V|}
\]

The top‑\( K \) tokens with the most negative gradient are selected as candidates, and greedy search performs the actual replacement.

---

## Experimental Setup

### Datasets

- **MT‑Bench**: 80 carefully crafted questions spanning 8 domains, each with 6 responses generated by different LLMs.
- **LLMBar**: 419 manually filtered QA pairs used to evaluate the judge’s instruction‑following capability.

The authors constructed a new evaluation dataset comprising 10 target questions, 10 target responses, and 500 clean responses.

### Target Response Generation

For each target question, they used GPT‑3.5‑turbo to generate a series of incorrect, illogical, malicious, or absurd responses and selected the most inappropriate one as the target.

### Clean Response Generation

Multiple LLMs were used to generate clean candidate responses, including GPT‑3.5‑turbo, GPT‑4, Gemma‑7B, Llama‑2 series, Mistral‑7B‑Instruct, Mixtral‑8x7B‑Instruct, Openchat‑3.5, and Claude‑2.

### Baselines

Manual attacks compared: naive attack, escape character, context ignoring, fake completion, combined attack, and others.

### Hardware

The paper does not specify exact hardware, but gradient‑based optimisation typically requires GPUs (e.g., NVIDIA A100/V100) for model inference and gradient computation.

---

## In‑Depth Analysis

### Why JudgeDeceiver Matters

This work is significant not merely for proposing a new attack, but for exposing a **systemic vulnerability** in the emerging LLM‑as‑a‑Judge paradigm.

First, **automation** raises the bar: manual injection relies on expert intuition and trial‑and‑error, while JudgeDeceiver enables one‑click generation of injection sequences. This dramatically lowers the barrier to launching effective attacks.

Second, the **threat model** is highly practical. The attacker only needs to know the target QA pair, the public instruction template, and the open‑source judge LLM – all of which are often publicly available in typical evaluation pipelines. The attacker does **not** need to know the other candidate responses, making the attack broadly applicable.

Third, the **three‑loss design** reflects a deep understanding of real attack requirements: effectiveness (\( \mathcal{L}_{aligned} \)), robustness (\( \mathcal{L}_{enhancement} \)), and stealthiness (\( \mathcal{L}_{perplexity} \)). These three dimensions are exactly what a practical attacker cares about.

### The Defence Dilemma

Perhaps the most alarming finding is that **none of the three existing defence classes can reliably stop JudgeDeceiver**. Known‑answer detection is completely ineffective; perplexity‑based filtering, while able to catch some cases, still misses 70% of attacks (at low false‑positive rates). This points to a deeper question: *When LLMs themselves become judges, how can we ensure they are not bribed?* No satisfying answer exists yet.

### Limitations and Future Directions

Currently, JudgeDeceiver focuses on **open‑source judge LLMs** because gradient information is required. For closed‑source APIs (e.g., GPT‑4), direct application is not possible. However, the optimisation framework is extensible – future work may explore black‑box optimisation or transfer‑based attacks to break this barrier.

---

## Practical Recommendations

### For LLM‑as‑a‑Judge Service Providers

1. **Input isolation and inspection** – Perform independent perplexity analysis and anomaly detection on each candidate response, especially those with stylistic deviations.
2. **Diversified judging** – Avoid relying on a single LLM; use ensemble voting or incorporate human spot‑checks.
3. **Position randomisation** – Shuffle the order of candidate responses in the input prompt to reduce positional dependence.
4. **Strict output formatting** – Constrain the judge’s output format to limit exploitation of the output template.
5. **Adversarial training** – Augment training/fine‑tuning data with adversarial examples similar to JudgeDeceiver to improve robustness.

### For Researchers

1. **Explore stronger defences** – The paper clearly shows that current defences are insufficient. Possible directions: semantic‑based anomaly detection, adversarial example detection, and better safety alignment of the judge LLM itself.
2. **Study transferability to closed‑source models** – Investigate how to launch similar attacks without gradient access, using query‑based optimisation or transfer from open‑source models.
3. **Build dedicated benchmarks** – Establish standardised security benchmarks for LLM‑as‑a‑Judge to facilitate systematic evaluation across the community.

---

## References

- Original paper: Shi, J., Yuan, Z., Liu, Y., Huang, Y., Zhou, P., Sun, L., & Gong, N. Z. (2024). JudgeDeceiver: Optimization‑Based Prompt Injection Attack to LLM‑as‑a‑Judge. *Proceedings of the ACM Conference on Computer and Communications Security (CCS 2024)*. [https://arxiv.org/abs/2403.17710](https://arxiv.org/abs/2403.17710)
- Official code repository: [https://github.com/ShiJiawenwen/JudgeDeceiver](https://github.com/ShiJiawenwen/JudgeDeceiver)
- PDF (including appendix): [https://arxiv.org/pdf/2403.17710](https://arxiv.org/pdf/2403.17710)
