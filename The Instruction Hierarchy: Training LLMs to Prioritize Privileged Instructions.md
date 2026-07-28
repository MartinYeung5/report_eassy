# The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions — Summary & Analysis

## Paper Highlights

This paper from OpenAI (April 2024) addresses a long‑overlooked but critical issue: current LLMs treat system prompts (developer instructions) and user inputs with equal priority—which is exactly why prompt injection, jailbreaking, and similar attacks succeed. The authors propose an **Instruction Hierarchy** framework, training models to selectively ignore lower‑priority instructions through novel data generation methods. Experiments on GPT‑3.5 show significant robustness gains against various attacks—including strong generalisation to attack types never seen during training—with almost no degradation in standard capabilities.

---

## Core Research Content

- **Problem Definition**  
  Modern LLMs receive multiple types of input in real‑world applications: system messages (from developers), user messages (from end users), and tool outputs (from third‑party sources). From an application‑design perspective, these inputs clearly should be treated differently. However, existing LLMs lack a mechanism to distinguish their priority, allowing attackers to use prompt injection to let low‑priority malicious instructions override high‑priority system‑level ones. The paper analogises this to an OS permission flaw—*“every instruction executes as if in kernel mode”*, letting third parties arbitrarily access private data and functions.

- **Innovative Approach**  
  The paper defines a clear three‑level hierarchy: system messages (highest), user messages (medium), and tool outputs (lowest). Two data generation strategies are designed:
  - **Context Synthesis** (for *aligned* instructions—those consistent with higher‑priority ones): decompose composite instructions into smaller pieces placed at different levels, training the model to predict the full original response.
  - **Context Ignorance** (for *misaligned* instructions—conflicting with higher‑priority ones): train the model to act as if it never saw those lower‑priority instructions.
  
  Specific training data are generated for prompt injection, system prompt extraction, and jailbreaking—but **jailbreaking data are deliberately withheld** from training to test generalisation.

- **Results**  
  Experiments on GPT‑3.5 Turbo with SFT and RLHF show:
  - Primary evaluation: robust gains; system‑prompt extraction defence improved by **63%**.
  - Generalisation tests (unseen jailbreaks, password extraction, tool‑use prompt injection): robustness improved by **34%**.
  - Standard capabilities (TriviaQA, LAMBADA, HellaSwag) remained essentially unchanged.
  - Over‑refusal evaluation: the model performed comparably to the baseline on most non‑conflicting instructions.

- **Practical Applicability**  
  This work offers a viable path for secure LLM deployment. For AI agents, email assistants, web‑browsing agents, etc., the hierarchy can prevent indirect prompt injection from stealing user data or performing unauthorised actions. The data generation is automated and scalable. Notably, OpenAI has since integrated this thinking into its Model Spec, confirming its practical value.

---

## Technical Details

### The Instruction Hierarchy

| Level | Source | Priority | Role |
|-------|--------|----------|------|
| System | Developer | Highest | Defines behaviour, safety constraints, tools |
| User | End user | Medium | Direct queries and requests |
| Tool | Third‑party | Lowest | Web search results, API outputs, etc. |

When multiple instructions coexist, lower‑priority ones may be *aligned* (consistent with higher‑priority goals) or *misaligned* (conflicting). For aligned instructions, the model should follow them; for misaligned ones, it should ignore or refuse.

### Data Generation Details

**Context Synthesis (for aligned instructions):**
1. Use an LLM to generate a composite instruction (e.g., “write a 20‑line poem in Spanish”).
2. Split it into fragments: “write a poem”, “use Spanish”, “use 20 lines”.
3. Place these fragments at different levels (system, user, tool).
4. Train the model to predict the original full response.

**Context Ignorance (for misaligned instructions):**
1. Generate system messages with various rules/constraints.
2. Generate user queries that try to trick the model into breaking those rules.
3. Train the model to predict the answer it would give *without* seeing the user instruction.

For closed‑domain tasks (e.g., summarisation), the paper assumes no aligned instructions—everything from the user is treated as data, not as an instruction.

### Training Strategies per Attack Type

| Attack Type | Training Strategy |
|-------------|-------------------|
| Direct prompt injection (open‑domain) | Context Synthesis + Context Ignorance |
| Direct prompt injection (closed‑domain) | Context Ignorance only |
| Indirect prompt injection | Context Ignorance for all instructions from browsing/tools |
| System prompt extraction | Refusal training for explicit extraction requests |
| Jailbreaking | **Intentionally excluded** from training (to test generalisation) |

---

## Experimental Setup

- **Model**: GPT‑3.5 Turbo
- **Training**: Supervised Fine‑Tuning (SFT) + RLHF
- **Baseline**: Same configuration without hierarchy‑aware training data
- **Selection**: Best checkpoint chosen by validation accuracy
- **Evaluation suite**:
  - In‑domain attacks (same distribution as training)
  - Generalisation tests (unseen jailbreaks, password extraction, tool‑use injection)
  - Over‑refusal (benign prompts that might be mistakenly rejected)
- **Capability benchmarks**: TriviaQA, LAMBADA, HellaSwag

All metrics are reported with one‑standard‑deviation error bars.

---

## In‑Depth Analysis

### Key Insight: From “No Privilege” to “Permission Levels”

The paper’s deepest contribution is framing the root cause as a *lack of instruction privilege*—a systemic architectural flaw. This shift from viewing prompt injection as mere prompt‑engineering to an OS‑level privilege analogy is critical. Just as operating systems separate kernel and user modes, LLMs need a hierarchy to prevent lower‑priority inputs from overriding high‑priority ones. This is not just rhetoric: the paper explicitly notes that SQL injection and command injection were solved by “never treating user input as privileged instructions”—a mature security pattern now ported to LLMs.

### The Elegance of Conditional Compliance

Rather than simply ignoring all low‑priority instructions (which would break usability), the method teaches the model to distinguish *aligned* from *misaligned* instructions. This conditional compliance is far smarter than previous brute‑force approaches (e.g., Chen et al. 2024, which ignored all user instructions). It preserves the user’s legitimate ability to give instructions while blocking malicious overrides.

### Generalisation: From Memorising Rules to Understanding Principles

The most impressive finding is that the model defends against unseen jailbreaks (34% gain) despite never being trained on jailbreak data. This shows the model has *internalised* the hierarchy principle rather than merely memorising attack patterns. This generalisation is vital for real‑world deployment, where attackers constantly invent new methods. Teaching the abstract concept of “instruction conflict” offers a degree of defence against unknown threats.

### Limitations and Challenges

The paper honestly acknowledges several limitations:
1. **Over‑refusal**: On two adversarial evaluation sets (system‑probing questions and benign prompts that look like jailbreaks), the model showed degraded over‑refusal behaviour. Boundary cases may cause incorrect rejection of safe requests.
2. **Continued adversarial threat**: The conclusion admits that *“current models may still be vulnerable to strong adversarial attacks”*. As Simon Willison commented, *“lowering the chance of an attacker finding a vulnerability just means they will try harder until they find one that works.”*
3. **High‑stakes applicability**: The paper calls for further research on whether LLMs can become robust enough for high‑risk agent applications. This suggests that while robustness is greatly improved, fully trustworthy autonomous agents are still a future goal.

---

## Practical Recommendations

### For LLM Application Developers

- **Restructure your inputs**: Place task instructions in the System Message; put user content and third‑party data in User or Tool messages. This simple step is the foundation for leveraging the hierarchy.
- **Add hierarchy‑aware data in fine‑tuning**: If you fine‑tune your own model, follow the paper’s data generation methods to include aligned/misaligned contrast samples. The approach is model‑agnostic.
- **Monitor over‑refusal**: Pay special attention to boundary prompts that look like attacks but are safe. Set up monitoring to catch over‑refusal patterns.

### For AI Safety Researchers

- **Hierarchy as a general framework**: The three‑level structure (system > user > tool) provides a clear research scaffold. Future work can explore more granular levels or dynamic priority adjustments.
- **Value of generalisation‑oriented evaluation**: The deliberate exclusion of jailbreaking data to test generalisation is a good example of “leave‑out” evaluation—true safety comes from understanding principles, not memorising attacks.
- **Combine with system‑level guardrails**: The hierarchy can be layered with other defences (e.g., external classifiers) to create a multi‑level security posture.

### Industry Outlook

OpenAI has already incorporated this concept into its Model Spec, signalling the transition from research to practice. For organisations building AI agents, enterprise chatbots, or automated workflows, adopting an instruction hierarchy can serve as a critical baseline security measure. However, as the paper concedes, for high‑risk autonomous agents, further advances are still required.

---

## References

- Original paper: Wallace, E., Xiao, K., Leike, R., Weng, L., Heidecke, J., & Beutel, A. (2024). The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions. *arXiv:2404.13208*. [https://arxiv.org/abs/2404.13208](https://arxiv.org/abs/2404.13208)

- Simon Willison’s commentary: *The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions* (April 23, 2024) [https://simonwillison.net/2024/Apr/23/the-instruction-hierarchy/](https://simonwillison.net/2024/Apr/23/the-instruction-hierarchy/)

- OpenAI Model Spec: [https://cdn.openai.com/spec/model-spec-2024-05-08.html](https://cdn.openai.com/spec/model-spec-2024-05-08.html)
