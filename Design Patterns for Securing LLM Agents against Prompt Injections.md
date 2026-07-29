
# An In-depth Analysis of "Design Patterns for Securing LLM Agents against Prompt Injections" (2025)

This analysis provides a systematic breakdown of the paper by Luca Beurer-Kellner et al., which introduces six architectural design patterns to establish *provable* defenses against prompt injection attacks in LLM agents.

## Paper Highlights

This paper systematically proposes six design patterns for building LLM agents, aiming to provide **provable defense capabilities** against prompt injection attacks. The core thesis is a paradigm shift: rather than attempting to build omniscient general-purpose agents and relying on heuristic detection, developers should **consciously constrain the agent's capability boundaries**. Through system-level isolation design, the paper achieves a quantifiable balance between security and utility.

## Core Research Content

### Problem Definition

Prompt injection is one of the most critical security threats facing LLM agents. When agents are granted tool-calling capabilities or need to process sensitive information, attackers can embed malicious instructions within user inputs or third-party data to manipulate the model into executing unauthorized operations. Existing defenses fall into three categories: LLM-level defenses (e.g., prompt engineering, adversarial training) lack theoretical guarantees; user-level defenses (e.g., human confirmation) severely undermine automation; and system-level isolation mechanisms, while promising, lack systematic design guidance. The core question this paper addresses is: **Given the current state of LLM technology, what kind of agents can we build that perform useful work while possessing verifiable resistance to prompt injection?**

### Innovative Methodology

The authors break away from the conventional mindset of "making the model more robust" and propose a fundamentally different path: **radically limiting the attack surface through system architecture design**. The core principle is straightforward—once an LLM agent ingests untrusted input, it must be constrained so that it is *impossible* for that input to trigger any consequential action with negative impact.

Based on this principle, the paper proposes six composable design patterns, ordered from strongest to weakest security guarantees:

1.  **Action-Selector Pattern**: The agent acts purely as a "translator," mapping natural language requests to a predefined, finite set of actions. The results of action execution are **not fed back** to the agent, making it naturally immune to prompt injection. Best suited for customer service bots.

2.  **Plan-Then-Execute Pattern**: The agent first formulates a **fixed tool-calling plan** based on the user instruction and then executes it. Untrusted data returned by tools **cannot** influence the agent's subsequent action choices, effectively implementing "control-flow integrity" protection. However, it cannot prevent injections within the user's initial prompt.

3.  **LLM Map-Reduce Pattern**: Decomposes tasks into "map" and "reduce" phases. Isolated sub-agents process independent third-party data fragments separately. Each sub-agent is strictly constrained (e.g., only outputting booleans), ensuring that even if compromised, the damage is contained.

4.  **Dual LLM Pattern**: Distinguishes between a privileged LLM (with tool access, but which never processes untrusted data) and an isolated LLM (which processes untrusted data but has no tool access). The isolated LLM's output is returned as a **symbolic reference**. The privileged LLM can only operate on these references without dereferencing them, completely blocking the injection pathway.

5.  **Code-Then-Execute Pattern**: The agent writes a formal computer program to solve the task, which can call tools and spawn sub-LLM instances. The program is executed on untrusted data only *after* the planning phase is complete, achieving complete separation between "planning" and "execution."

6.  **Context-Minimization Pattern**: Proactively removes unnecessary contextual content during multi-turn interactions. For instance, after converting a user request into a database query, the original user prompt is removed before execution to prevent injection from affecting subsequent responses.

### Research Outcomes

The paper validates these design patterns through **ten cross-domain case studies**, covering scenarios such as OS assistants, SQL agents, email/calendar assistants, customer service bots, booking assistants, product recommendation, resume screening, medical Q&A, diagnostic chatbots, and software engineering agents. Each case is analyzed across three levels: "naive implementation → applied design patterns → security vs. utility trade-offs."

Key findings include:
- **General-purpose agents cannot provide reliable security guarantees in the foreseeable future**, as the adversarial robustness problem of LLMs remains unsolved after over a decade of research.
- By **consciously limiting the agent's scope of tasks**, provable security can be achieved without severely compromising utility.
- The design patterns are **composable**—different scenarios require different security-utility trade-off strategies.

### Practical Application Potential

These design patterns are not theoretical constructs but **architectural guidelines directly deployable in production**:
- Customer service bots can directly adopt the Action-Selector pattern.
- File-processing agents can leverage the Map-Reduce pattern.
- Scenarios requiring complex reasoning can adopt the Dual LLM or Code-Then-Execute patterns to maintain flexibility while isolating untrusted data.

The authors, affiliated with institutions like ETH Zurich, Google, Microsoft, and IBM, base their research on real-world application scenarios, offering high engineering reference value.

## Technical Details

### Core Mechanisms

**Symbolic References** are a critical technology in the Dual LLM pattern. After the isolated LLM processes untrusted text, instead of returning the content directly, it stores the result in a special variable (e.g., `$VAR`). The privileged LLM can operate on these variables symbolically (e.g., calling `send.email($VAR)`), but **cannot dereference** them to view the content. The actual dereferencing is performed by a non-LLM "Orchestrator" at function-call time. This fundamentally prevents injected content from ever entering the privileged LLM's context.

**Control-Flow Integrity** is the theoretical foundation of the Plan-Then-Execute pattern. The agent "commits" to a fixed sequence of tool calls before execution. Subsequently, any data originating from untrusted sources cannot alter this sequence. This mirrors traditional security concepts—even if data is tainted, the control flow remains intact.

### Formalization of Security Guarantees

The security guarantees described in the paper can be formalized as follows:

Let the agent's action space be \(A\), untrusted input be \(x\), and system prompt be \(s\). Without applying design patterns:

\[
\text{Action}(x) \subseteq A \quad \text{(Attackers can induce arbitrary actions)}
\]

After applying the Action-Selector pattern:

\[
\text{Action}(x) \subseteq A_{\text{pred}} \subset A \quad \text{(Actions are restricted to a predefined subset)}
\]

After applying the Plan-Then-Execute pattern:

\[
\text{Plan}(s) = P, \quad \text{Execute}(P, x) \text{ does not alter } P \quad \text{(Plan is fixed, execution is not affected by injection)}
\]

### Technical Details from Case Studies

**OS Assistant**:
- *Naive*: Full shell access → arbitrary code execution via injection.
- *Action-Selector*: Hardcoded command set → completely immune, but loses flexibility.
- *Plan-Then-Execute*: Fixed command sequence → prevents out-of-plan commands, but parameters can still be manipulated.
- *Dual LLM/Map-Reduce*: Orchestrator schedules isolated sub-agents to process files one by one; sub-agent outputs constrained to booleans or predefined categories → single malicious file compromises at most the result for that specific file.

**SQL Agent**:
- Database content is treated as an **untrusted source**.
- Adopts **Code-Then-Execute**: SQL queries and Python analysis code are generated and executed, preventing the LLM from directly processing query results.
- Combined with **Action-Sandboxing**: Python code execution must occur within a sandbox.
- Combined with **Ask-The-User**: Seeks user confirmation before critical operations.

## Research Setup

### Experimental Design

The paper does not rely on quantitative experiments (like attack success rates). Instead, it employs a **case study methodology**: for ten real-world application scenarios, it systematically analyzes the design space across "naive implementation → applied design patterns → security vs. utility trade-offs." Each case clearly defines:
- Application scenario and functional requirements.
- Threat model (which inputs the attacker controls, attack objectives).
- Various design schemes and their security-utility trade-offs.

### Hardware/Software Configuration

The paper focuses on architecture-level design and is not dependent on specific hardware.
- **LLM**: Model-agnostic; the patterns apply to any instruction-tuned LLM.
- **Tools**: Agents interact with external systems via Agent-Computer Interfaces.
- **Orchestrator**: A non-LLM software component responsible for tool call scheduling and symbolic reference dereferencing.
- **Sandbox**: For isolated code execution.

## Comprehensive Analysis

### Core Insight: From "Model Security" to "System Security"

The most fundamental contribution of this paper lies in its **paradigm shift**: it reframes prompt injection from a "model robustness problem" to a "system architecture problem." The authors astutely point out that after more than a decade, adversarial examples remain unsolved in computer vision; expecting LLMs themselves to be robust against prompt injection is unrealistic. The real solution lies in **system-level design**—even if the underlying model is compromised, the overall agent system remains secure.

This insight has profound implications. It means security researchers should not invest all their energy into "preventing the model from being fooled," but rather think about "how to design systems so that even if the model is fooled, it doesn't matter"—the latter is an engineering-solvable problem.

### Quantifying the Security-Utility Trade-off

The value of this paper lies not only in proposing design patterns but also in **systematically analyzing the security-utility trade-off for each pattern**. From Action-Selector (most secure, least flexible) to Code-Then-Execute (more flexible, security dependent on implementation quality), it forms a complete design spectrum. Developers can choose the appropriate pattern based on their specific scenario rather than being forced into a binary choice between "completely insecure" and "completely unusable."

### Comparison with Existing Approaches

Existing defenses are mostly heuristic—input/output filters, adversarial training, anomaly detection, etc. These approaches "raise the bar for attacks" but provide no guarantees. In contrast, the security guarantees from this paper's patterns are **architectural**: not because the model "learned" to reject injections, but because the system design makes injections *structurally incapable* of triggering dangerous actions.

### Limitations and Open Problems

The paper does not shy away from the limitations:
- **Action-Selector** pushes most of the work to designing predefined actions, which may negate the flexibility advantages of LLMs.
- **Plan-Then-Execute** cannot prevent injections from manipulating tool parameters (e.g., sending an email to an attacker instead of the intended recipient).
- **Map-Reduce** requires tasks to be decomposable into independent subtasks, which doesn't apply to all scenarios.
- **Dual LLM** adds significant system complexity via symbolic references.
Furthermore, the paper explicitly states these patterns **cannot solve injections originating from the user's initial prompt**—Context-Minimization only offers partial mitigation against malicious users.

## Practical Application

### Decision-Making Guide

| Scenario Characteristics | Recommended Pattern(s) | Rationale |
| :--- | :--- | :--- |
| Limited, pre-enumerable action space (e.g., Customer Service) | **Action-Selector** | Complete immunity to injection; simple to implement. |
| Tasks pre-planable without adjusting based on intermediate results | **Plan-Then-Execute** | Control-Flow Integrity guarantee. |
| Processing large amounts of independent data fragments (e.g., Batch File Processing) | **Map-Reduce** | Isolates single points of failure; enables parallel processing. |
| Processing untrusted text while maintaining high flexibility | **Dual LLM** | Symbolic references completely isolate the injection path. |
| Complex tasks requiring code generation | **Code-Then-Execute** | Complete separation of planning and execution. |
| Multi-turn conversational scenarios | **Context-Minimization** | Used in conjunction with other patterns. |

### Implementation Recommendations

1.  **Start with Threat Modeling**: Clearly define which inputs are under the attacker's control (user input? third-party data? database content?) and the potential attack objectives (data exfiltration? code execution? privilege escalation?).
2.  **Default to the Principle of Least Privilege**: The agent's tool access permissions should be strictly limited to the minimum set required to complete the task.
3.  **Combine Design Patterns**: As demonstrated in the SQL Agent case, combine Code-Then-Execute (isolating data from the LLM), Action-Sandboxing (isolating code execution), and Ask-The-User (human confirmation for critical operations).
4.  **Distinguish between Trusted and Untrusted Data Flows**: The core principle is—**privileged components must never process untrusted data, and components processing untrusted data must never hold privileges**.
5.  **Use "Best Practices" as a Baseline**: The general best practices mentioned in Appendix A of the paper (sandboxing, user confirmation, etc.) should serve as baseline security measures for all LLM agents.

## References

- Original Paper: https://arxiv.org/abs/2506.08837
- PDF Full Text: https://arxiv.org/pdf/2506.08837
- Authors: Luca Beurer-Kellner, Daniel Dobos, Kathrin Grosse, Beat Buesser, Daniel Fabian, Daniel Naeff, Ana-Maria Crefu, Marc Fischer, Ezinwanne Ozoani, Vaclav Volhejn, Andrew Paverd, Florian Tramer, Edoardo Debenedetti, David Froelicher
- Submitted: June 10, 2025 (v1), Last revised June 27, 2025 (v3)
