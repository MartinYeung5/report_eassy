
# Analysis of "Autonomy Reshapes How Personalization Affects Privacy Concerns and Trust in LLM Agents" (2026)

> **Paper**: *Autonomy Reshapes How Personalization Affects Privacy Concerns and Trust in LLM Agents* (2026)  
> **Authors**: Zhiping Zhang, Yi Evie Zhang, Freda Shi, Tianshi Li  
> **Institutions**: Northeastern University, University of Illinois Urbana‑Champaign, University of Waterloo  
> **arXiv**: [2510.04465](https://arxiv.org/abs/2510.04465)

---

## 📌 Key Points

LLM‑based agents require access to vast amounts of personal data to deliver personalised services, but this inevitably raises users’ privacy concerns and undermines trust. Through a 3×3 between‑subjects experiment with **N=450**, this study demonstrates that **risk‑contingent autonomy** – where the agent cedes control back to the user upon detecting potential privacy leaks – effectively alleviates the negative effects of personalisation on privacy concerns and trust by enhancing users’ **perceived control**.

---

## 🧠 Core Research

### Problem Definition

LLM agents (e.g., OpenClaw, OpenAI Operator, Claude Code) possess far greater autonomy than traditional non‑agentic AI systems. To deliver effective personalisation, they need to connect to external applications (Gmail, Notion, etc.) and access user data. However, unlike traditional AI that merely generates static content, LLM agents plan and execute actions in real time – for example, sending messages to specific recipients through real‑world tools. This interaction with external environments introduces data leakage risks.

This creates a fundamental dilemma: **personalisation needs data, data sharing triggers privacy concerns, and privacy concerns inhibit data sharing** – thereby limiting both the agent’s autonomy and the effectiveness of personalisation. The authors call this the **“personalisation‑privacy dilemma”**.

Yet the expanded design space of agent autonomy may also offer new opportunities to reshape these effects – an aspect that has remained unexplored in prior work.

### Innovative Approach

The core innovation lies in **systematically introducing agent autonomy as a moderator** into the personalisation‑privacy dilemma framework, and proposing **“risk‑contingent autonomy”** as a novel interaction design paradigm.

Specifically, building on the autonomy framework of Kasirzadeh and Gabriel (2025), the authors constructed a **3×3 between‑subjects experiment (N=450)**:

**Three autonomy levels:**
- **No autonomy**: Every agent action requires user approval.
- **Intermediate autonomy**: The agent executes routine operations autonomously, but pauses and requests user confirmation when detecting risks (e.g., accessing sensitive data).
- **Full autonomy**: The agent performs all actions independently.

**Three personalisation types:**
- **No personalisation**: The agent does not access any user data.
- **Privacy‑aware personalisation**: The agent accesses only non‑sensitive user information.
- **Full personalisation**: The agent can access all user data from connected applications.

The study measured **privacy concern**, **trust**, and **willingness to use** as core dependent variables, and examined **perceived sensitivity** and **perceived control** as mediating mechanisms to uncover how autonomy moderates the effects of personalisation.

### Key Findings

1. **The double‑edged sword of personalisation**  
   Personalisation that does not respect users’ privacy preferences significantly increases privacy concerns and decreases trust and willingness to use. This effect is mediated by **perceived sensitivity** – the more sensitive users perceive the accessed data, the higher their privacy concerns and the lower their trust and willingness to use.

2. **The moderating role of autonomy**  
   Autonomy significantly moderates these effects. Specifically, **intermediate autonomy** (compared to no autonomy and full autonomy) **flattens** the impact of personalisation – it attenuates the increase in privacy concerns and the decrease in trust caused by personalisation. Intermediate autonomy has a significant negative effect on privacy concerns and a significant positive effect on trust.

3. **Effectiveness of risk‑contingent autonomy**  
   The most critical finding: **risk‑contingent autonomy alleviates the negative effects of personalisation by enhancing perceived control**. When the agent cedes control back to the user upon detecting potential privacy leaks, users feel a stronger sense of control, which reduces privacy worries and sustains trust in the agent.

4. **Clarification of mediation mechanisms**  
   Personalisation affects user responses via **perceived sensitivity**, but intermediate autonomy does **not** significantly affect perceived sensitivity or its downstream outcomes. This means the moderating effect of autonomy operates through a **separate pathway – perceived control** – rather than by changing users’ perception of data sensitivity.

### Practical Applicability

The results offer immediate and actionable value:

- **Interaction design guidelines** for LLM agents – providing evidence‑based principles to achieve a win‑win between personalisation and privacy through risk‑contingent autonomy.
- **Privacy UX design paradigm** – a concrete interaction pattern where the agent dynamically assesses operation risk and proactively “brakes” when sensitive data is involved, rather than pursuing full automation.
- **Trust calibration tool** – helping enterprises maintain user trust by giving users a sense of control without sacrificing personalisation effectiveness.
- **Compliance support** – offering a technical approach to meet regulatory requirements (GDPR, CCPA) regarding user control.

---

## 🔧 Technical Details

### Experimental Design

A standard 3×3 factorial between‑subjects design, with participants randomly assigned to one of nine condition combinations. The scenario involved a hypothetical LLM agent assistant connected to the user’s email, calendar, contacts, and purchase history.

### Variable Operationalisation

**Independent variables:**
- **Personalisation type** – manipulated by varying the scope of data the agent can access:
  - No personalisation: no user data accessed.
  - Privacy‑aware: only non‑sensitive data (e.g., calendar events).
  - Full: all information (email content, chat logs, purchase history, etc.).
- **Autonomy level** – based on Kasirzadeh and Gabriel (2025):
  - No autonomy: every action requires approval.
  - Intermediate: routine actions autonomous; pauses and asks for confirmation when risk is detected.
  - Full: all actions fully autonomous.

**Dependent variables:**
- Privacy concern – measured with established scales.
- Trust – assessed across competence, reliability, and benevolence dimensions.
- Willingness to use – intention to continue using the agent.

**Mediators:**
- Perceived sensitivity – how sensitive the user perceives the accessed data.
- Perceived control – the degree to which the user feels they can control the agent’s behaviour.

### Analytical Methods

**ANOVA** to test main and interaction effects, and **mediation analysis** to examine the mediating roles of perceived sensitivity and perceived control. Detailed hypothesis test results are available in Table 1 and Table 4 of the paper.

---

## 🧪 Study Setup

### Participants

- **Sample size**: N = 450
- **Design**: 3 (personalisation) × 3 (autonomy) between‑subjects
- **Data collection**: Online survey platform (e.g., Prolific or similar)

### Procedure

Participants read a scenario describing an LLM agent, including its functions, accessible data types, and autonomy level. They were then asked to imagine using the agent for daily tasks (scheduling meetings, replying to emails, managing calendars, etc.) and completed the relevant questionnaires.

### Hardware/Software Requirements

As a user experiment, the study primarily relied on:
- Online survey tools (Qualtrics, Prolific)
- Standard statistical software (SPSS, R, or Python)
- No special hardware required.

---

## 🔍 Comprehensive Analysis

### Theoretical Contributions

This paper makes several important theoretical contributions:

1. **Expands the concept of “autonomy” in HCI** – previous work focused on the *degree* of autonomy, whereas this study introduces the *type* of autonomy, particularly **conditional autonomy**. The finding that more autonomy is not always better (full autonomy can exacerbate privacy concerns) and less is not always better (no autonomy feels burdensome) highlights the importance of **the right autonomy at the right time**.

2. **Reveals the central role of perceived control** in the personalisation‑privacy dilemma. While prior work emphasised perceived sensitivity as the mediator, this study shows that autonomy’s moderating effect does **not** operate through perceived sensitivity. Instead, **perceived control** is the key psychological mechanism, offering a more precise target for design interventions.

3. **Theorises the specificity of LLM agents** – unlike non‑agentic AI, LLM agents can plan and execute actions in real time, so autonomy is not just about “automation” but about “under what conditions automation occurs”. The study captures and theorises this distinction.

### Practical Implications

The most important practical takeaway is: **designing agent autonomy is essentially about designing for *human autonomy***. This involves two dimensions:
- **Perceived control**: making users feel they can influence the agent’s behaviour.
- **Monitoring effectiveness**: enabling users to effectively oversee the agent’s actions.

Together, these form the foundation of user trust in the agent.

### Limitations and Future Directions

- The experiment used a **hypothetical scenario** rather than real agent usage.
- Interaction was **simulated**, lacking longitudinal data from extended use.
- Focused on **personal use**; multi‑user or organisational contexts were not examined.
- Future work could validate these findings in real‑world settings and explore cross‑cultural differences in reactions to autonomy and privacy.

---

## 💼 Practical Applications

### For Product Managers

- **Don’t equate autonomy with full automation** – fully autonomous agents may reduce trust. Design “safety valves” at critical decision points, especially when sensitive data is involved.
- **Risk‑contingent autonomy is a viable design pattern** – equip the agent with runtime risk detection and give control back to the user when potential privacy leaks are detected.
- **Perceived control matters more than actual control** – even if the agent runs autonomously most of the time, as long as users believe they can intervene when necessary, trust can be sustained.

### For Developers

- **Implement runtime risk detection** – the agent must assess risk before performing sensitive operations (e.g., accessing private emails, sending messages on behalf of the user).
- **Design clear transparency interfaces** – let users know what data is being accessed, why, and how it is used.
- **Provide granular control options** – allow users to set different autonomy preferences for different data types and operations.

### For Researchers

- **Include perceived control as a key mediator** in future HCI and AI trust studies.
- **Explore other types of conditional autonomy** – beyond risk, are there other conditions (uncertainty, complexity, etc.) where conditional autonomy is beneficial?
- **Conduct longitudinal studies in real‑world environments** to validate the long‑term effects of risk‑contingent autonomy.

---

## 📚 References

- Original paper: *Autonomy Reshapes How Personalization Affects Privacy Concerns and Trust in LLM Agents* (2026)  
  - arXiv: [https://arxiv.org/abs/2510.04465](https://arxiv.org/abs/2510.04465)  
  - PDF: [https://arxiv.org/pdf/2510.04465](https://arxiv.org/pdf/2510.04465)  
  - HTML: [https://arxiv.org/html/2510.04465v2](https://arxiv.org/html/2510.04465v2)  
- Primary categories: cs.HC (Human‑Computer Interaction), cs.AI (Artificial Intelligence), cs.CR (Cryptography and Security)  
- Submitted: 6 Oct 2025 (v1), last revised: 6 Apr 2026 (v2)
```
