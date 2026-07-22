
# In-depth Analysis of "Personalized Safety in LLMs: A Benchmark and A Planning-Based Agent Approach"

> **Paper Info**: Yuchen Wu, Edward Sun, Kaijie Zhu, Jianxun Lian, Jose Hernandez-Orallo, Aylin Caliskan, Jindong Wang | NeurIPS 2025 | arXiv:2505.18882

---

## Key Contributions

This paper is the first to systematically define **Personalized Safety** for large language models, exposing the fundamental flaw of one-size-fits-all alignment in high‑stakes applications—the same "safe" response can carry vastly different risks for different users. The authors construct the **PENGUIN** benchmark with 14,000 scenarios and propose **RAISE**, a training‑free two‑stage agent framework. Across six mainstream LLMs, RAISE improves safety scores by up to **31.6%**, while requiring only **2.7 user queries** on average to collect personal context.

---

## Core Research

### Problem Definition

Current LLM safety evaluations rely heavily on context‑agnostic metrics—factual accuracy, toxicity scores, or bias measures. This paradigm assumes a universal "safe" standard for all users, but reality is far more nuanced: a medical dosage recommendation that is safe for an average adult may be lethal for a pregnant woman or a patient with specific conditions; a financial tip harmless to a risk‑tolerant investor could be disastrous for a retiree. The paper identifies this underexplored issue as **Personalized Safety**—safety assessment and assurance must be grounded in the user’s specific background, condition, and vulnerabilities.

### Innovative Approach

The paper contributes on two levels:

**Level 1: The PENGUIN Benchmark**  
PENGUIN is the first benchmark designed specifically for personalized safety, containing 14,000 scenarios across seven sensitive domains. Each scenario includes both **context‑rich** and **context‑free** variants to precisely quantify how much user background information influences safety judgments. This dual‑variant design allows researchers to measure the true value of "knowing who the user is."

**Level 2: The RAISE Framework**  
RAISE (**R**e**A**ctive **I**nformation **S**election & r**E**sponse) is a training‑free, plug‑and‑play two‑stage agent. Its core philosophy: rather than collecting all possible user information (impractical and privacy‑invasive), it strategically selects the most critical background details. The two stages are:
- **Information Acquisition**: The agent proactively asks targeted questions to obtain the most relevant context with minimal interaction cost.
- **Response Generation**: It tailors the LLM output to match the user’s specific safety needs based on the collected information.

### Key Results

Systematic evaluation on six mainstream LLMs yielded:

| Metric | Value |
|--------|-------|
| Safety improvement from personalization | **43.2%** |
| Additional improvement from RAISE | **Up to 31.6%** |
| Average RAISE queries per user | **2.7** |

Importantly, not all contextual attributes contribute equally—this validates RAISE’s design of strategic information selection.

### Practical Deployment Potential

RAISE’s "no‑retraining" nature makes it highly practical. It can be deployed as a **safety wrapper** on top of existing LLM applications without expensive fine‑tuning. Use cases include:
- **Medical/health assistants**: adjust safety margins based on patient age, history, or medications.
- **Financial services**: tailor investment advice safety filters to user risk tolerance.
- **Educational tools**: adapt content appropriateness to student age and cognitive level.
- **Mental health apps**: tune response sensitivity to emotional state and psychological history.

---

## Technical Details

### PENGUIN Scenario Design

The seven sensitive domains (inferred from the paper) cover: healthcare, finance, law, psychology, education, political/social issues, and personal safety. Each scenario follows:
- **Dual‑variant control**: both a user‑background‑rich version and a background‑free version.
- **Automated evaluation pipeline**: carefully designed for reliable assessment.
- **Scale**: 14,000 scenarios for statistically significant conclusions.

### RAISE Two‑Stage Mechanism

RAISE operates as a planning‑based agent:

**Stage 1: Strategic Information Acquisition**
- The agent analyzes the query to identify which user background factors are most critical for safety.
- It asks minimal questions (average 2.7) to obtain the most relevant data.

**Stage 2: Personalized Safe Response**
- Dynamically adjusts safety criteria based on acquired user context.
- Generates a response aligned with the user's specific safety requirements.
- No parameter updates to the underlying LLM.

This cleverly transforms personalization from a training problem into an inference‑time retrieval and integration problem, drastically lowering deployment barriers.

---

## Experimental Setup

### Evaluation Configuration

- **Models**: Six mainstream LLMs (including both closed‑source and open‑source, specified in the paper).
- **Benchmark**: PENGUIN with 14,000 scenarios.
- **Domains**: Seven sensitive areas, each with context‑rich and context‑free variants.

### Hardware and Software Requirements

Since RAISE is training‑free, hardware needs match those of running the base LLM—no extra training clusters or large storage. Software requires:
- Environment for LLM API calls or local inference.
- Implementation of the RAISE two‑stage logic.
- Optional: PENGUIN evaluation pipeline.

---

## Comprehensive Analysis

### Academic Significance

This paper fundamentally challenges the prevailing paradigm of LLM safety. Prior work—RLHF, red‑teaming, safety fine‑tuning—has all aimed at a *universal* safety standard. The paper exposes a sharp paradox: if "safety" is user‑dependent, then pursuing a single standard is a false goal.

The "Planning‑Based Agent" framing is particularly insightful—it recasts safety as a sequential decision‑making problem, opening new research avenues.

### Limitations and Future Directions

Critical points to consider:
1. **Privacy–Safety tension**: personalization requires sensitive user data, introducing new risks not fully explored.
2. **User cooperation assumption**: average 2.7 queries may still be burdensome; real‑world willingness to engage needs validation.
3. **Dynamic user states**: risk profiles evolve over time, but RAISE appears as a one‑time acquisition without continuous tracking.
4. **Domain representativeness**: while 14,000 scenarios is large, the selection and coverage of seven domains could be further justified.

---

## Practical Recommendations

### For Researchers

- **Adopt PENGUIN** as a standard evaluation tool for personalized safety research to enable comparable assessments.
- **Extend RAISE** by exploring more efficient acquisition strategies or finer‑grained adaptation mechanisms.

### For Practitioners

- **Rapid prototyping**: RAISE can be wrapped around existing LLM apps with minimal effort—test personalized safety impact quickly.
- **Cost control**: 2.7 queries per interaction is manageable, but balance personalization depth with latency for your specific use case.
- **Privacy compliance**: ensure adherence to GDPR, CCPA, etc.—consider adding differential privacy or local processing to RAISE.

### For Product Decision‑Makers

- **Differentiation**: personalized safety can be a competitive edge, especially for high‑risk domains like healthcare, finance, and education.
- **User trust**: explaining why you collect certain information and how it improves safety can strengthen user confidence.

---

## References

- Original Paper: Wu, Y., Sun, E., Zhu, K., Lian, J., Hernandez‑Orallo, J., Caliskan, A., & Wang, J. (2025). *Personalized Safety in LLMs: A Benchmark and A Planning‑Based Agent Approach*. NeurIPS 2025. [arXiv:2505.18882](https://arxiv.org/abs/2505.18882)
- NeurIPS Official Page: [https://neurips.cc/virtual/2025/loc/san-diego/poster/118931](https://neurips.cc/virtual/2025/loc/san-diego/poster/118931)
- Hugging Face Papers: [https://huggingface.co/papers/2505.18882](https://huggingface.co/papers/2505.18882)
