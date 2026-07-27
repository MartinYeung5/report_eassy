
# Privacy Leakage Overshadowed by Views of AI: A Study on Human Oversight of Privacy in Language Model Agent — In‑depth Analysis (CHI 2025)

This is a comprehensive analysis and summary of the CHI 2025 paper:  
**“Privacy Leakage Overshadowed by Views of AI: A Study on Human Oversight of Privacy in Language Model Agent”**  
Original paper: [https://arxiv.org/abs/2411.01344](https://arxiv.org/abs/2411.01344)

---

## Key Points

This paper is the **first empirical work** that systematically investigates humans’ ability to supervise privacy risks in Language Model (LM) Agents. Through a task‑based online survey with 300 participants, the study reveals a striking finding: **users tend to prefer AI‑generated responses that leak more private information over their own drafted replies**, causing the rate of harmful disclosure to jump from 15.7% (self‑drafted) to 55.0% (after choosing AI replies). Furthermore, a clustering analysis identifies **six distinct privacy behaviour patterns**, exposing the heterogeneity in how users oversee agent actions.

---

## Core Research Content

### Problem Definition

As LM Agents (e.g., OpenAI’s Operator, AgentGPT) gain increasing autonomy to perform personal tasks on behalf of users—such as replying to emails or composing social media posts—they can access private user data (calendars, contacts, etc.). However, this autonomy introduces novel privacy risks: agents may inadvertently disclose sensitive information without explicit user consent. The paper cites a concrete case where an LM Agent, after accessing John’s calendar data, revealed in an email to John’s manager that John “is in talks with several companies about job changes.”

Prior work mainly focused on measuring privacy leakage at the model level and aligning models to reduce it, but **no one has yet examined whether human users are actually capable of effectively overseeing an agent’s privacy risks**. The core research question is: *When an AI acts on our behalf, can we really tell when it is leaking our private information?*

### Innovative Methods

**Task‑based survey design** – a carefully structured four‑stage online survey:

1. **Draft response**: Participants first write their own reply to one of six scenarios (from the PrivacyLens dataset, covering personal life updates, job searches, psychological counselling, travel planning, etc.).
2. **Introduction of LM Agent**: Participants are introduced to the concept and capabilities of LM Agents, and asked about their trust and comfort in delegating the specific task to an agent.
3. **Preference selection**: Participants read the agent‑generated reply and choose between “my draft”, “agent’s reply”, or “both equally good”.
4. **Privacy evaluation**: After being informed of the Privacy Tuples in the scenario, participants rate the harmfulness of the disclosed information.

**Multi‑dimensional analysis** – combining quantitative and qualitative approaches:
- **Objective privacy leakage**: defined based on pre‑labelled sensitive information.
- **Subjective privacy leakage**: based on participants’ own assessment of harmfulness.
- **Qualitative coding**: bottom‑up coding of participants’ reasoning and concerns about LM Agents.
- **Clustering analysis**: HDBSCAN non‑parametric clustering based on five behavioural dimensions to identify user patterns.

### Research Findings

**Key quantitative results:**

| Metric | Value |
|--------|-------|
| Sample size | N = 300 |
| Preference for agent reply or “both” | 48.0% |
| Harmful disclosure rate in self‑drafts | 15.7% |
| Actual harmful disclosure rate after choice | 38.4% – 55.0% |
| Number of privacy behaviour clusters identified | 6 |

The core phenomenon—**“privacy risk overshadowed by the AI halo”**—is clearly demonstrated: even when some participants’ choices objectively reduce leakage, it is often coincidental rather than intentional.

**Six privacy behaviour patterns** – identified via HDBSCAN clustering (silhouette score 0.43), revealing significant differences in privacy concerns, trust in AI, and privacy preferences.

**Dynamic trust changes** – certain groups showed a notable decrease in trust and comfort after completing the task.

### Potential for Real‑World Application

1. **Privacy design guidelines** for AI agent products (email assistants, schedule managers, social media helpers, etc.), grounded in empirical evidence.
2. **Trust calibration mechanisms** – the proposed concept of “bidirectional privacy preference alignment” can be directly used to design interaction patterns that help calibrate user trust.
3. **Personalised privacy protection** – the six identified behaviour patterns provide a basis for user segmentation and adaptive privacy safeguards.
4. **Human‑AI collaboration frameworks** – empirical support for more effective “human‑in‑the‑loop” oversight mechanisms.

---

## Technical Details

### Privacy Leakage Measurement Framework

The study defines privacy leakage based on **Contextual Integrity (CI) theory**, operationalising it as “the LM Agent’s behaviour violates contextual privacy norms.”

**Privacy Tuples** consist of three key dimensions:
- **Actors**: sender (always the user), data subject (user’s own information vs. others’ information), and recipient (specific vs. general).
- **Data Types**: personal information (identity or experiences) vs. professional information (workplace data, often with explicit confidentiality).
- **Transmission Principles**: the normative constraints governing information flow.

### Clustering Implementation

**Feature selection (five dimensions)**:
1. Number of information items rated as harmful by the participant (binarised: harmless → 0, harmful → 1).
2. Participant’s reply preference.
3. Privacy concern level.
4. Trust in LM Agent.
5. Individual subjective leakage rate (calculated as |H|/|S|, where H is the set of pre‑labelled items rated harmful, and S is all pre‑labelled sensitive items).

**Algorithm choice**: Compared k‑means, agglomerative, DBSCAN, and HDBSCAN; ultimately used **HDBSCAN** (non‑parametric) for its ability to find clusters of varying densities with minimal tuning. Optimal parameters: `min_samples=5`, `min_cluster_size=20`, Euclidean distance.

### Qualitative Coding Framework

Participants’ reasoning was coded into 16 codes grouped into 4 themes:
- **Privacy‑related**
- **Usefulness**
- **Expression**
- **Personality**

---

## Experimental Setup

| Aspect | Details |
|--------|---------|
| Study type | Task‑based online survey |
| Sample | N = 300 |
| Participant criteria | Located in the U.S., aged ≥18 |
| Scenarios | 6 scenarios from PrivacyLens dataset (personal life, job search, counselling, travel planning, etc.) |
| LM Agent model | SOTA LLMs (e.g., GPT‑4) |
| Survey platform | Qualtrics |

**Quality control measures**:
- Copying of scenario materials and pasting into response fields were disabled to prevent AI‑generated drafts.
- Excluded responses with completion time <5 minutes (average ~13 min).
- Excluded participants who rated scenarios as “irrelevant” or “neutral”.
- Manual screening of low‑quality data.

**Hardware/software requirements**:
- Survey platform: Qualtrics
- AI model: GPT‑4 (or equivalent) to generate agent replies
- Scenario data: synthetic user data and agent replies from PrivacyLens
- Analysis tools: clustering (HDBSCAN, k‑means, etc.) and qualitative coding software

---

## Comprehensive Analysis

### Academic Contributions and Theoretical Significance

**Filling a critical gap.** This is the first empirical study to systematically examine humans’ ability to oversee privacy risks in LM Agents. Prior work largely focused on model‑level privacy metrics and alignment, overlooking the most essential link—whether the final decision‑maker (the human user) can effectively supervise. The findings sharply demonstrate that even if model‑level protections are robust, user‑side failures can nullify those efforts.

**Challenging the naive “human‑in‑the‑loop” assumption.** AI safety research often assumes that keeping humans in the loop guarantees safety. This paper empirically disproves that: 48% of participants chose agent replies that were objectively more privacy‑invasive. The result challenges the entire paradigm—**merely being “in the loop” is insufficient; we need to ensure humans are *effectively* in the loop.**

**Revealing subjectivity and heterogeneity in privacy.** The study finds significant variability in how participants judge the harmfulness of the same information. This supports the view that “privacy is subjective and ambiguous,” and questions current practices that use LLM‑as‑a‑Judge for privacy assessment—if even human experts cannot agree on privacy norms, how can LLMs reliably judge them?

### Implications for AI Product Design

**The “AI halo effect” warning.** Users tend to overestimate the quality of AI‑generated content and are willing to accept higher privacy risks for perceived convenience or professionalism. Product designers must establish **explicit visualisation mechanisms for privacy risks** to counteract blind trust.

**Need for trust calibration.** Observed trust declines after the first encounter with a privacy‑sensitive event suggest that **early negative experiences can cause lasting trust damage**. Products should proactively guide users to understand privacy boundaries during the first use, rather than remediating after incidents.

### Limitations

- The drafting task provided additional cognitive scaffolding that may not exist in real‑world use.
- Scenarios, though diverse, are still limited in coverage.
- Participants were all from the U.S.; cross‑cultural generalisability remains unverified.

---

## Practical Applications

### For AI Agent Product Developers

1. **Design a “Privacy Impact Statement”**: Before any operation that involves sharing personal information, clearly explain to users—in plain language—what will be shared, with whom, and what the potential impact is.
2. **Implement “bidirectional privacy preference alignment”**: Not only should the agent learn user preferences, but users should also understand the agent’s decision logic, creating mutual cognitive alignment.
3. **Layered oversight mechanisms**: Based on the six behaviour patterns, offer different levels of supervision—from full autonomy to step‑by‑step confirmation—allowing users to choose according to their privacy sensitivity.
4. **Visual privacy risk comparison**: Intuitively compare the privacy impact of “what the user would do” vs. “what the agent plans to do” to help users make more informed choices.

### For Enterprises Deploying AI Agents

1. **Employee training**: Before deploying AI email assistants or scheduling tools, train employees on privacy risk awareness to prevent unintentional disclosure due to the “AI halo.”
2. **Privacy audit logs**: Maintain complete audit trails of all agent operations, especially those involving personal information, to enable post‑hoc tracing and risk assessment.
3. **Gradual authorisation**: Start with a “watch mode” where employees observe agent behaviour before granting higher levels of permission.

### For End Users

1. **Maintain critical thinking**: Do not assume AI‑generated replies are more professional or safer—the AI does not know your privacy boundaries.
2. **Actively review sensitive information**: Before authorising the agent to send any communication involving personal or others’ information, manually check for any unintended sharing.
3. **Create a personal privacy checklist**: Clearly identify types of information you never want to share (e.g., health, finances, job changes) and stay vigilant when using AI agents.

---

## References

- Original paper: [https://arxiv.org/abs/2411.01344](https://arxiv.org/abs/2411.01344)
- PDF full text: [https://arxiv.org/pdf/2411.01344](https://arxiv.org/pdf/2411.01344)
- CHI 2025 – ACM Conference on Human Factors in Computing Systems
