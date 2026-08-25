
# LLM Fingerprinting via Semantically Conditioned Watermarks

## Key Points

This paper introduces a new paradigm for large language model (LLM) fingerprinting that replaces the fragile "fixed query–key pair" design with a combination of **semantic-domain queries** and **statistical watermark signals**. Experiments show that the proposed method achieves **100% detection rate** under common deployment scenarios such as fine‑tuning, quantization, and pruning, while remaining stealthy. It is the first black‑box fingerprinting scheme that simultaneously provides robustness and covertness.

---

## Core Contributions

### Problem Definition

Training state‑of‑the‑art LLMs is extremely expensive. When model owners release open‑source weights, they typically attach restrictive licenses (e.g., prohibiting commercial use). However, malicious third parties may ignore these terms and deploy the model behind a commercial API. Owners therefore need a reliable way to prove ownership — i.e., to uniquely identify their model from its output responses.

Existing black‑box fingerprinting methods rely on making the model "memorise" specific abnormal responses (keys) to a small set of fixed queries. This dependency has two critical flaws:

1. **Fragility**: Common non‑adversarial deployment operations like fine‑tuning, quantization, and pruning often erase the fingerprints.
2. **Lack of stealth**: The abnormal queries and responses are easily detected and filtered by a malicious deployer.

### Innovative Approach

The core innovation is a fundamental paradigm shift:

**(i) Replacing fixed query sets with a semantic domain**

Instead of relying on a few fixed queries, the method uses an entire **semantic domain** as the fingerprint trigger — for example, "any French‑language query" will activate the fingerprint signal. Even if the input is perturbed (e.g., by changing the system prompt), the perturbed input usually still falls within the same semantic domain, so the fingerprint remains accessible.

**(ii) Replacing fixed key tokens with a statistical watermark**

Instead of making the model "memorise" specific token sequences, the method embeds a **statistical signal** distributed across the entire generated response — the longer the response, the stronger the signal. During detection, the owner issues multiple queries from the semantic domain, concatenates the responses, and runs a statistical test.

**(iii) First‑ever single‑domain conditional watermark embedding**

By leveraging Red‑Green watermarking and watermark distillation, the method successfully embeds the watermark **only in generations from a single chosen semantic domain**. The model's output distribution remains unchanged for non‑target domains, ensuring that normal usage is unaffected.

### Results

Extensive experiments validate the approach:

| Evaluation Aspect | Result |
|-------------------|--------|
| Robustness (fine‑tuning / quantisation / pruning / sampling variation) | **100%** detection rate |
| Adversarial attacks (5 targeted attacks) | All defeated – fingerprint persists |
| Stealthiness | No significant quality degradation in the target domain; non‑target domains completely unaffected |
| False positive control | Statistically guaranteed via hypothesis testing |

### Practical Deployment Potential

The technology is applicable in several real‑world scenarios:

- **Model copyright protection**: Owners can prove that a suspicious API service uses their model, providing technical evidence for legal actions.
- **Usage monitoring**: Track where and how the model is deployed, assisting business or security decisions.
- **Open‑source licence compliance**: Ensure that users of open‑source models adhere to licence terms (e.g., non‑commercial clauses).

---

## Technical Details

### 1. Red‑Green Watermark Mechanism

At each generation step, using a private key ξ and the previous k tokens (context), the vocabulary Σ is pseudo‑randomly partitioned into **green** tokens (fraction γ) and **red** tokens (fraction 1−γ). The logits of green tokens are boosted by δ, making them more likely to be sampled. For detection, a one‑sided z‑test is performed on the number of green tokens in the given sequence to determine whether the watermark is present.

### 2. Semantic Conditioned Watermark Distillation

The model owner first duplicates the original model θ₀ as an immutable reference. Then, on the target semantic domain D_target, the watermark is distilled by minimising the KL divergence between the logits of model θ and those processed by the Red‑Green watermark:

$$L_{\text{watermark}}(\theta, \xi)(x) = \sum_{t=1}^{|x|} KL\bigl(\text{Red-Green}(p_{\theta_0}(\cdot \mid x_{<t})),\; p_{\theta}(\cdot \mid x_{<t})\bigr)$$

At the same time, a regularisation term keeps the model's distribution unchanged on non‑target domains:

$$L_{\text{reg}}(\theta) = KL\bigl(p_{\theta_0}(\cdot \mid x_{<t}),\; p_{\theta}(\cdot \mid x_{<t})\bigr), \quad x \notin D_{\text{target}}$$

### 3. Fingerprint Detection Workflow

The model owner selects multiple queries from the target semantic domain (e.g., French), sends them to the suspect model, concatenates all responses, and runs the watermark detection test. Because the watermark signal increases with the number of tokens, multiple queries amplify the detection power.

### 4. Choice of Semantic Domain

The paper notes that the target semantic domain should have **high entropy** (i.e., the model's output distribution averaged over the domain has high entropy) to ensure sufficient watermark signal. It should also be **restricted enough** to prevent an adversary from easily detecting the fingerprint. The main experiments use **French** as the target domain, with other domains explored in the appendix.

---

## Experimental Setup

### Models and Baselines

- **Models fingerprinted**: Mainstream open‑source LLMs (the paper does not restrict to a specific architecture).
- **Baselines**:
  - **Instructional Fingerprinting (IF)**
  - **Scalable Fingerprinting (SF)**

### Deployment Scenarios Simulated

The following common deployment operations are systematically evaluated:

| Operation | Description |
|-----------|-------------|
| Fine‑tuning | Additional training on task‑specific datasets |
| Quantisation | Representing weights and activations with lower precision |
| Pruning | Zeroing out unimportant weights |
| Sampling variation | Different decoding and sampling hyperparameters |

### Adversarial Attacks

Five targeted adversarial attacks are tested, including query filtering and output filtering strategies.

### Stealthiness Evaluation

The paper compares the utility (text quality) of the fingerprinted model on target vs. non‑target domains, and also checks whether an adversary can detect the presence of the fingerprint.

---

## In‑Depth Analysis

### Paradigm Shift: From "Memory" to "Behavioural Signature"

Traditional fingerprinting essentially exploits the LLM's **memory** — forcing it to memorise a few specific question‑answer pairs. The problem is that memory is fragile: fine‑tuning may overwrite it, quantisation may corrupt it, and a malicious deployer can easily filter the exact strings.

This work shifts the core idea from *what the model remembers* to **how the model generates** — under a certain semantic condition, the model's generative behaviour exhibits a statistically verifiable bias. This is analogous to identifying a person by their handwriting style rather than by whether they can write a specific word. Such an approach is more fundamental and much harder to evade.

### The Elegance of Semantic Domain Selection

Choosing "French" as the target domain is a clever design decision:

- **Natural stealth**: French queries are not uncommon in normal usage, so an attacker cannot easily distinguish "normal French replies" from "fingerprinted French replies".
- **High entropy guarantee**: The richness of French provides ample "space" for embedding the watermark signal.
- **Domain‑wide robustness**: No matter how the user varies the French phrasing, as long as it stays within the French domain, the fingerprint is triggered.

This "area‑based" rather than "point‑based" design is a major advantage in adversarial settings — an attacker might block a few specific questions, but blocking an entire semantic domain without degrading the model's functionality is nearly impossible.

### Implications for the Open‑Source Ecosystem

If widely adopted, this technique will have profound impact on the open‑source LLM ecosystem. Currently, open‑source models are frequently misused in commercial API services, and copyright holders often struggle to provide evidence. With a robust fingerprint that is nearly impossible to remove, owners can "implant" a digital fingerprint before release, and later simply ask a few French questions to a suspicious API to perform forensics.

Of course, this also raises an important question: could the technology be misused for over‑restrictive control over open‑source models? Technology is neutral; its boundaries must be defined by law and ethics.

### Limitations and Future Directions

The paper honestly discusses potential limitations. First, although black‑box fingerprinting is more practical than white‑box methods, detection still requires API interaction. Second, the choice of semantic domain involves a trade‑off — too broad a domain reduces stealth, while too narrow a domain increases the number of queries needed for detection. In addition, alternative identification methods such as timing attacks deserve further exploration.

---

## Practical Recommendations

### For Model Providers

1. **Timing of fingerprint injection**: It is recommended to perform watermark distillation right before final release, after the model has completed training, to minimise any impact on its original capabilities.

2. **Semantic domain selection strategy**:
   - Prefer **high‑entropy** languages (e.g., French, German, or other morphologically rich languages) to ensure sufficient signal.
   - Avoid overly broad domains (e.g., "English") to preserve everyday usability and stealth.
   - Tailor the domain to the target market — for example, a European‑facing model might use a smaller European language.

3. **Detection strategy**:
   - When a violation is suspected, select multiple queries covering diverse topics within the target domain.
   - Concatenate responses and run the statistical watermark test.
   - If the signal from a single round is weak, increase the number of queries to amplify it.

4. **Key management**: Safeguard the private key ξ used for the Red‑Green watermarking — it is the core credential for detection.

### For Model Users

If your organisation uses open‑source LLMs and plans to deploy them commercially, carefully review the licence terms. This technology gives model owners a much stronger means to track and prove usage, and the legal risks of unauthorised commercial use are rising.

---

## References

- Original paper: [LLM Fingerprinting via Semantically Conditioned Watermarks](https://arxiv.org/html/2505.16723) (arXiv:2505.16723v3, ETH Zurich)
