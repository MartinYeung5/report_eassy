# Trust, Risk, and Security in Agentic AI: A Comprehensive Analysis

## Paper Highlights
This survey systematically tackles the core challenges of trust, risk, and security management in LLM-based Agentic Multi-Agent Systems (AMAS). By extending the Gartner TRiSM framework to this novel paradigm, the authors identify emerging threats unique to multi-agent collaboration and introduce two quantitative evaluation metrics. This work establishes a critical theoretical foundation and offers practical guidelines for the safe and trustworthy deployment of agentic AI.

---

## Core Research Content

### Problem Definition
Traditional AI security frameworks are designed for single-model or static systems. In stark contrast, agentic AI systems consist of dynamic collaborative networks of multiple LLM-driven agents—each equipped with tool-calling capabilities, memory mechanisms, and autonomous decision-making authority to pursue long-term goals. This architectural paradigm shift renders conventional trust, risk, and security management approaches largely obsolete. The core question this paper addresses is: as AI evolves from "answering questions" to "taking autonomous actions," how do we fundamentally redefine and ensure its trustworthiness, safety, and controllability?

### Innovative Approaches
The paper's primary contribution lies in the systematic adaptation of Gartner's AI TRiSM framework to meet the unique demands of agentic multi-agent systems. The framework is structured around five core pillars:

- **Explainability**: Rendering the collaborative decision-making processes of multiple agents traceable and comprehensible.
- **ModelOps**: Covering full lifecycle management, from development to deployment.
- **Application Security**: Defending against diverse attacks targeting agent applications.
- **Model Privacy**: Protecting the confidentiality of model parameters and training data.
- **Lifecycle Governance**: Establishing robust governance mechanisms throughout the system's entire lifespan.

Furthermore, the paper proposes two novel quantitative metrics:
- **Component Synergy Score (CSS)**: Quantifies the quality of collaboration and mutual empowerment among multiple agents.
- **Tool Utilization Efficacy (TUE)**: Evaluates the accuracy and efficiency of agents' tool invocation within operational workflows.

The authors also establish a comprehensive risk taxonomy for AMAS, systematically mapping emerging threats such as prompt injection, agent collusion, and memory poisoning.

### Research Outcomes
As a survey paper, its core "findings" are not experimental datasets but rather a systematic intellectual framework and analytical toolkit:

1.  **Conceptual Clarification**: Clearly delineates the fundamental architectural differences between agentic AI and traditional AI agents.
2.  **Framework Construction**: Maps the five TRiSM pillars onto AMAS workflows and failure modes, creating actionable evaluation dimensions.
3.  **Threat Modeling**: Establishes a risk taxonomy covering scenarios like coordination failures and adversarial prompt manipulation.
4.  **Quantitative Tools**: Provides two practically implementable metrics—CSS and TUE—for system-level assessment.

### Potential for Practical Deployment
The frameworks and metrics proposed in this paper offer clear engineering value. CSS and TUE can be seamlessly integrated into the CI/CD pipelines of agentic AI systems as quantitative quality gates. The five TRiSM pillars provide enterprises with a structured self-assessment checklist—offering actionable touchpoints for everything from compliance audits to security testing, privacy protection to operational monitoring. Notably, Gartner has identified Agentic AI and TRiSM among its top strategic technology trends for 2025, underscoring the strong industry recognition of this framework.

---

## Technical Details

### Adapting the TRiSM Framework for AMAS
A core technical insight of this paper is that security issues in agentic AI are not isolated points of failure but systemic challenges. The five pillars are not independent; they form an integrated chain of trust:
