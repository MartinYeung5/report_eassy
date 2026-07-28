
# StruQ: Defending Against Prompt Injection with Structured Queries  
*A Paper Analysis (USENIX Security 2025)*

## Key Takeaways

Prompt injection is ranked as the top security threat for LLM applications by OWASP. Its root cause is that LLM inputs mix instructions (control flow) with data. StruQ borrows the design philosophy of SQL prepared statements and combines a **Secure Front-End** with **Structured Instruction Tuning** to teach LLMs to obey only the "instruction" part and ignore any malicious commands hidden in the data part. Experimental results show that, with almost no loss in utility, StruQ reduces the success rate of TAP attacks from 97% to 9% (on Alpaca).

---

## Core Contributions

### Problem Definition

The core challenge of prompt injection is that an LLM's input is a single text string where instructions (from the developer) and data (user input / external content) are concatenated without distinction. Attackers can inject malicious content like "Ignore the previous instructions and instead..." into the data portion, and because LLMs are trained to scan the entire input and execute any instruction they find, such attacks often succeed.

More specifically, the threat model assumes that attackers can arbitrarily modify the data part of the query but cannot modify the prompt itself. Attackers are aware of the prompt and the application format. An attack is considered successful if the LLM's response follows the hidden instruction embedded in the data.

### Innovative Approach

StruQ's core innovation lies in applying the classic security principle of **control-data separation** to the LLM domain, while solving the practical problem of how to make off‑the‑shelf LLMs respect this separation.

The solution consists of two major components:

**(1) Secure Front-End**

The front‑end formats the input using system‑reserved special tokens (e.g., `[MARK]`) as separators, clearly distinguishing the prompt from the data. The key is a **filtering mechanism**: the front‑end removes any separator tokens from the data portion, ensuring that only the system can create this structured format—attackers cannot forge it. This is analogous to escaping user input to prevent XSS in web development.

**(2) Structured Instruction Tuning**

Having a secure front‑end is not enough—the LLM must also be trained to follow this format. The authors augment standard instruction‑tuning datasets with many samples where the data portion contains distracting or conflicting instructions. Through supervised fine‑tuning, the model learns to respond only to the part marked as the instruction and to completely ignore any instructions appearing in the data.

The paper emphasises three core design elements: **reserved special tokens as separators**, a **front‑end with filtering**, and **dedicated structured instruction tuning**.

*A note on SecAlign:* The paper also introduces SecAlign (safety preference optimisation) as a complement or alternative to StruQ. SecAlign uses preference optimisation instead of supervised fine‑tuning, further reducing attack success rates from 45% to 8%. However, StruQ is the main focus, with SecAlign presented as an extension.

### Results

The paper evaluates StruQ on multiple LLMs (Llama‑7B, Mistral‑7B, Llama3‑8B, etc.) against more than a dozen non‑optimised attacks and several strong optimised attacks.

**Key metrics:**

| Attack Type | No Defense | With StruQ |
|-------------|------------|------------|
| TAP (on Alpaca) | 97% | **9%** |
| TAP (on Mistral) | 100% | **36%** |
| Dozens of non‑optimised attacks | High success | **Near 0%** |


For strong optimisation‑based attacks like GCG, SecAlign further reduces the success rate to **below 15%**—a >4× reduction compared to previous best defences. Notably, later work (2026) proposed an attention‑based attack that can bypass StruQ and SecAlign with up to 70% success, indicating that the field is evolving rapidly; StruQ significantly raises the bar but is not the final solution.

In terms of utility, StruQ has negligible impact on model performance—the loss on AlpacaEval is within about one standard error.

### Practical Deployment Potential

**High applicability**, for several reasons:

1. **Generality**: A single LLM fine‑tuned with StruQ can be used for any downstream task without per‑task retraining.
2. **Affordable cost**: It fine‑tunes existing models, far cheaper than pre‑training from scratch (which costs millions of dollars).
3. **No extra manual annotation**: Training data can be generated automatically.
4. **Comprehensive protection**: It defends against a wide range of attacks, from simple injections to sophisticated TAP.

---

## Technical Details

### Structured Query Format

A structured query consists of two independent components:
- **Prompt**: the trusted instruction from the application developer.
- **Data**: untrusted content from the user or external sources.

The secure front‑end encodes the query into a special format using a hard‑coded template (based on standard formats like Alpaca). Critically, the front‑end filters out all reserved separator tokens from the data, ensuring that attackers cannot inject them to forge an instruction region.

### Training Data Construction

The core of structured instruction tuning is how training data is built:

1. **Normal samples**: standard instruction‑response pairs where only the prompt contains an instruction.
2. **Adversarial samples**: random "distractor instructions" (e.g., "Ignore the previous instruction...") are injected into the data part, and the model is trained to still respond only to the prompt's instruction.

Through this process, the model learns to "only obey what is marked as the instruction, ignoring all instructions in the data."

### Attack Types Covered

The paper systematically evaluates several attack categories:

- **Naive Attack**: simple injection of extra instructions.
- **Ignore Attack**: injection of "ignore previous instructions" and its variants.
- **Escape Character Attacks**: using special characters like `\b`, `\r`, `\n`, `\t` to attempt to overwrite or separate text.
- **Completion Attacks**: first appending a fake "response complete" marker and then injecting malicious instructions.
- **TAP (Tree‑of‑Attacks with Pruning)**: originally a multi‑LLM collaborative attack for jailbreaking, adapted here for prompt injection.

---

## Experimental Setup

### Models and Datasets

- **Base models**: Llama‑7B, Mistral‑7B, Llama3‑8B, etc.
- **Fine‑tuning dataset**: augmented from standard instruction‑tuning datasets like Alpaca.
- **Utility evaluation**: AlpacaEval.

### Attack Evaluation Settings

- Attackers are assumed to **know the prompt and the application format**, but cannot modify the prompt.
- For each attack type, **multiple variants** are tested (e.g., 10 variants of Ignore Attack).
- For Completion attacks, three cases are distinguished: using legitimate separators (Completion‑Real), approximate separators (Completion‑Close), and irrelevant separators (Completion‑Other).

### Training Cost

The paper emphasises that StruQ's training cost is **far lower than pre‑training an LLM from scratch** (which costs millions of dollars). It is essentially instruction fine‑tuning on an existing pre‑trained model, with computational overhead comparable to standard instruction tuning.

---

## In‑Depth Analysis

### Security Perspective

StruQ's elegance lies in transplanting a **fundamental security principle**—control‑data separation—to the LLM world. Looking back at the history of computer security, the solution to SQL injection was prepared statements, to XSS was output encoding/input filtering, and to command injection was parameterisation—all sharing the same core idea. StruQ does essentially the same: transforming the LLM's input from an "unsafe single‑string API" into a "secure structured‑query API." This "standing on the shoulders of giants" design gives StruQ strong methodological credibility.

### Practical Limitations

However, we must also be realistic about several issues:

**First**, the defence is not absolute. Subsequent work (2026) has demonstrated an attention‑based architecture‑aware attack that can bypass StruQ with up to 70% success. This shows that the cat‑and‑mouse game continues; StruQ is an important step, but not the final word.

**Second**, deployment requires changes. StruQ requires deploying a secure front‑end on the application side and fine‑tuning the LLM accordingly—for systems already in production and difficult to replace, the migration cost is non‑negligible.

**Third**, can the front‑end's filtering mechanism be "bulletproof" in practice? Any flaw in the filtering logic could be exploited by attackers, necessitating rigorous formal verification or extensive adversarial testing.

### Academic Contributions

From a research perspective, this paper makes several noteworthy contributions:

1. **Precise problem diagnosis**: it identifies the two root causes—lack of control/data separation in the input, and LLMs being trained to execute any instruction in the input.
2. **Systematic solution**: instead of patching, it rethinks the input format and model training holistically.
3. **Comprehensive evaluation**: covering attacks from simple to complex, manual to automated.
4. **Open‑source and reproducible**: code and project page are publicly available.

---

## Practical Recommendations

### Suitable Scenarios

Consider StruQ if your LLM application:

- **Handles external/user‑provided data** (e.g., document analysis, web summarisation, API results).
- **Has high security requirements** (e.g., enterprise apps, sensitive data).
- **Aims to boost security without significantly sacrificing performance**.

### Deployment Advice

**1. Start with the front‑end**: Even without fine‑tuning, deploying the secure front‑end (using separators and filtering user input) can already raise the bar against some attacks.

**2. Assess your attack surface**: Identify which parts of your application are vulnerable to injection—user comments, uploaded documents, scraped web content, etc.

**3. Consider phased rollout**:
- Phase 1: Collect data from production traffic to assess current injection risk.
- Phase 2: Deploy StruQ in a test environment and validate its impact on business functionality.
- Phase 3: Gradually roll out to production with canary releases.

**4. Stay updated**: StruQ is a 2025 technique, and attack methods are still evolving. Keep an eye on follow‑up improvements (like SecAlign) and the latest academic research.

---

## References and Resources

- Original paper: https://arxiv.org/abs/2402.06363
- Project website: https://sizhe-chen.github.io/StruQ-Website
- Code repository: https://github.com/Sizhe-Chen/StruQ
- USENIX Security 2025 conference page: https://www.usenix.org/conference/usenixsecurity25/presentation/chen-sizhe
- BAIR blog post: https://bair.berkeley.edu/blog/2025/04/11/prompt-injection-defense
