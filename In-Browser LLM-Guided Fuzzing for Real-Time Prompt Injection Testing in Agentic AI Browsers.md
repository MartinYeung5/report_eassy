
# In-Browser LLM-Guided Fuzzing for Real-Time Prompt Injection Testing in Agentic AI Browsers

**Paper:** [arXiv:2510.13543](https://arxiv.org/abs/2510.13543)  
**Authors:** Avihay Cohen  
**Category:** Cryptography and Security (cs.CR); Artificial Intelligence (cs.AI)

---

## Key Points

This paper presents a fully in‑browser fuzzing framework driven by an LLM to automatically discover prompt injection vulnerabilities in agentic AI browsers. Although all tested AI browsers and assistant extensions successfully block simple attacks, after ten rounds of adaptive mutation even the best‑performing tool fails at a rate of **58%–74%**. This finding exposes the fundamental weakness of static defenses against dynamically generated LLM attacks.

---

## Core Research Contributions

### Problem Definition

With the integration of LLMs into web browsers—creating “agentic AI browsers”—users can rely on AI assistants to automatically summarise pages, fill forms, click links, and more. However, this introduces a critical security threat: **indirect prompt injection attacks**.

Malicious instructions can be hidden in web content—e.g., invisible text, HTML comments, CSS‑obfuscated directives—which are invisible to humans but can be parsed and executed by the LLM. Worse, AI agents operate with the user’s privileges across sites, bypassing traditional web security mechanisms (same‑origin policy, sandboxing). Real‑world cases already exist: the Perplexity AI browser (Comet) was found vulnerable to hidden instructions that could leak private information and execute unauthorised cross‑site actions.

Existing testing methods rely mainly on manual work or simple template‑based systems, which are insufficient to cover the infinite variants of prompt injection attacks. Therefore, the core problem addressed is: **How to build an automated, high‑fidelity, adaptive fuzzing framework that can continuously evolve with defensive measures to discover prompt injection vulnerabilities?**

### Innovative Methodology

The paper introduces four key innovations:

1. **In‑browser fuzzing architecture.** Traditional fuzzing runs in isolated environments, failing to simulate the full DOM context and event handling of real browsers. This framework runs directly inside the target browser, achieving **high‑fidelity testing**—malicious pages are rendered, the AI agent processes them under realistic conditions, and the tester fully monitors DOM operations and agent behaviour.

2. **LLM‑driven attack generation and mutation.** A hybrid strategy combines **seed templates** (248 templates covering 12 attack categories) with **LLM‑based mutation**. The core novelty: the attack‑generation LLM receives feedback from previous rounds (success/failure, agent behaviour patterns) and produces new variants accordingly. The attack evolves—it “learns” and adapts over time.

   For example, the LLM might suggest splitting an instruction across meta descriptions and HTML comments, encoding directives in URL components (query params, fragments, path segments, domain), using HTML character entities, or embedding instructions in images for OCR‑based extraction. The mutation space is vast and far exceeds static templates.

3. **Real‑time feedback‑driven closed‑loop optimisation.** After each round, the framework collects immediate feedback (whether the agent executed hidden instructions) and uses it to guide the next generation. A strategy of **30% exploration and 70% exploitation** balances known effective patterns with new variants.

4. **Zero‑false‑positive detection.** The paper claims **zero false positives** in detection, which is critical for practical security testing tools—false alarms would otherwise waste significant manual review effort.

### Research Findings

Key experimental results:

- **Progressive evasion of defenses.** All tested agentic AI browsers and extensions block simple attacks (initial templates). However, as the LLM generates increasingly complex variants based on feedback, defenses degrade sharply—by the 10th fuzzing iteration, even the best tool fails **58%–74%** of the time. This reveals a fundamental issue: **static defenses are inherently unable to cope with dynamically evolving attacks**.

- **High‑risk features identified.** Two functionalities are particularly vulnerable: **page summarisation** (73% success rate) and **Q&A** (71% success rate). Both fully ingest all page content (including hidden directives) and users tend to trust the generated outputs. This provides clear priority guidance for developers.

- **Stronger generation models yield stronger attacks.** Using more powerful models (GPT‑4, Claude) for mutation reduces the **time‑to‑first‑success** by **3.3×** compared to smaller open‑source models (LLaMA). This implies an asymmetric landscape: defenders relying on weaker models may fail against attackers using top‑tier LLMs.

- **Framework capabilities.** The framework supports multiple backends (LLaMA 3.1/3.3 70B via Ollama, OpenAI, Claude, and custom endpoints) and can test various AI browser implementations. The full platform is open‑source for researchers and developers.

### Real‑World Application Potential

- **Continuous security testing & CI/CD integration.** The framework can be deployed in the development cycle of AI browser products, running automated regression tests before each release—especially valuable for rapidly iterating AI products.

- **Red‑teaming and security assessment.** Security teams can use it for systematic red‑teaming, quantitatively evaluating prompt injection defenses. The high‑risk features (summarisation, Q&A) serve as priority test vectors.

- **Defense validation.** When implementing new defenses (input filtering, output monitoring, instruction boundary detection, etc.), developers can verify their effectiveness against LLM‑generated dynamic attacks—not just static templates.

- **Benchmarking.** The framework can act as an industry benchmark to compare the security posture of different AI browsers, promoting higher standards.

- **Open‑source ecosystem.** The public release allows the community to build upon and improve the tool, fostering a more robust security testing ecosystem.

---

## Technical Details

### System Architecture

Four interconnected components form a closed‑loop test environment:

1. **Malicious page generator** – creates HTML pages with attack payloads (template‑based or LLM‑mutated).
2. **Browser automation module** – loads pages in a real browser.
3. **Agent monitoring module** – observes agent behaviour and output, detecting whether hidden instructions were executed.
4. **Feedback‑driven adaptation module** – analyses results and guides the next round of attack generation.

### Attack Generation Formulation

The payload generation can be expressed as:

$$P(t) = G_{LLM}(S_{templates}, F_{history}, C_{context})$$

where $G_{LLM}$ is the generation LLM, $S_{templates}$ the seed template set, $F_{history}$ historical feedback, and $C_{context}$ contextual information.

### Evaluation Metrics

| Metric | Definition |
|--------|------------|
| **Success Rate** | $ \frac{|\mathcal{S}|}{N} $, where $\mathcal{S}$ is the set of successful attacks and $N$ total trials |
| **Time‑to‑First‑Success** | $ \min_{p \in \mathcal{S}} t(p) $ |
| **Attack Diversity** | $ \frac{1}{|\mathcal{S}|} \sum_{p_i, p_j \in \mathcal{S}, i \neq j} d_{edit}(p_i, p_j) $, with $d_{edit}$ the normalised edit distance |
| **Detection Precision** | $ \frac{TP}{TP + FP} $ — the paper claims FP = 0 |

### Attack Vector Examples

The framework explores a wide range of vectors:

- **Cross‑element splitting** – splitting instructions between meta descriptions and HTML comments.
- **URL encoding injection** – encoding directives in query parameters, fragments, path segments, or domain names.
- **Character entity obfuscation** – using HTML character entities.
- **Multimodal attacks** – embedding instructions in images relying on OCR.

---

## Experimental Setup

### Hardware and Software

| Component | Configuration |
|-----------|---------------|
| Browser | Chrome / Chromium |
| CPU | Apple M4, 96 GB unified memory |
| Local LLM | Ollama (LLaMA models) |
| Network | 1 Gbps (for cloud LLMs like Claude, GPT‑4) |

### Fuzzing Parameters

| Parameter | Value |
|-----------|-------|
| Total iterations per model | N = 100 |
| Exploration rate | ε = 0.3 (30% explore, 70% exploit) |
| Temperature | α = 2.0 (moderate preference for successful templates) |
| Feedback window | k = 5 (last 5 attempts) |
| Timeout per test | T = 30 seconds |
| Template corpus | 248 (12 attack categories) |

### Compatible Agent Types

- Full agentic browsers (standalone AI‑integrated browsers)
- Browser extensions (Chrome/Edge AI assistants)
- Custom AI agents (any browser‑based content‑processing system)

---

## In‑Depth Analysis

### Methodological Contribution

This work represents a paradigm shift in fuzzing. Traditional fuzzing relies on input generation with mutation based on syntactic rules or genetic algorithms, followed by execution and crash observation. Here, the **LLM serves as the mutation engine**, leveraging semantic understanding to craft attack variants. This is no longer bit‑flipping—it is **semantic‑level attack evolution** where the LLM understands what constitutes an effective prompt injection and invents more deceptive variants.

The implications are profound: when defenders also use LLMs for security, the game becomes **LLM vs. LLM**. The finding that stronger models produce stronger attacks (3.3× faster success) confirms that test effectiveness depends not only on framework design but also on the **underlying generator’s capability**.

### Security Implications: The End of Static Defenses

The paper’s core warning: **static defenses inevitably fail against dynamic attacks**. From round 1 to round 10, success rates jump from near 0% to 58%–74%—a qualitative change, not just quantitative. This demands a new design philosophy for AI browser security: **defenses must be dynamic and adaptive**. Potential directions include real‑time suspicious pattern detection, behavioural monitoring of agent outputs, and least‑privilege execution.

### Privacy and Trust Dimensions

The high risk of summarisation (73%) and Q&A (71%) reveals a trust paradox: the more users trust AI outputs (summaries, answers), the more easily they can be misled; the more powerful the AI (full page ingestion), the larger the attack surface. Product design must balance intelligence with security—e.g., pre‑filtering page content, secondary validation of outputs, and user confirmation for high‑risk actions.

### Open‑Source and Ecosystem Impact

Releasing the framework publicly is a significant contribution. The AI security field lacks standardised testing tools; this can become a benchmark. However, attackers could also use it—a classic dual‑use dilemma. Nonetheless, in the long run, **public testing tools favour defenders** by lowering the bar for security testing and driving proactive vulnerability discovery.

---

## Practical Recommendations

### For AI Browser Developers

- **Prioritise high‑risk features.** Allocate extra defenses to summarisation and Q&A—e.g., input sanitisation, output monitoring, and user confirmation for sensitive operations.
- **Integrate continuous fuzzing.** Add the framework to CI/CD pipelines to test each release against both known and unknown (LLM‑generated) attacks.
- **Design dynamic defenses.** Given that static protections break within 10 rounds, consider adaptive filtering, anomaly detection, and human‑in‑the‑loop for suspicious actions.

### For Security Researchers

- **Extend test coverage.** Expand beyond indirect injection to direct injection, multi‑turn dialogue, and cross‑modal attacks (text + images).
- **Evaluate defensive strategies.** Use the framework to benchmark various defenses—input filters, output monitors, instruction boundary detectors, adversarial training—and identify which truly resist LLM‑driven dynamics.
- **Establish industry benchmarks.** Build standardised test suites, metrics, and rankings based on this methodology to increase transparency and drive continuous improvement.

### For End Users

- **Use summarisation cautiously.** This feature is highest‑risk; be especially careful with untrusted pages.
- **Stay updated.** Install security updates promptly as AI browser security evolves rapidly.
- **Understand agent privileges.** The AI acts with your permissions—avoid granting more authority than necessary.

---

## References

- Original paper: [https://arxiv.org/abs/2510.13543](https://arxiv.org/abs/2510.13543)  
- PDF version: [https://arxiv.org/pdf/2510.13543](https://arxiv.org/pdf/2510.13543)  
- 37 pages, 10 figures, 8 tables
