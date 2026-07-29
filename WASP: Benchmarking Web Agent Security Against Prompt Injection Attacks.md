
# WASP: Benchmarking Web Agent Security Against Prompt Injection Attacks

> *Accepted as a NeurIPS 2025 Poster paper.*

## Key Insights

WASP is the first end‑to‑end benchmark for evaluating the security of web agents against prompt injection attacks, proposed by Meta Research. The study reveals that even state‑of‑the‑art models (including those with advanced reasoning capabilities) can be deceived by simple, low‑cost prompt injections in highly realistic scenarios. Attacks achieve **partial success in up to 86%** of cases, yet complete success remains rare. This uncovers a previously overlooked phenomenon: current web agent security largely relies on *incompetence* rather than robust defense – the agents simply fail to fully execute complex attacker instructions.

---

## Core Research

### Problem Definition

Autonomous UI agents promise to boost human productivity by automating tasks like tax filing and bill payments. However, their ability to act on behalf of users introduces a critical security concern: **prompt injection attacks**, where adversaries embed malicious instructions into web content, tricking the agent into executing attacker‑defined actions.

Existing security evaluations for web agents suffer from systematic flaws: they either oversimplify the threat (testing unrealistic scenarios or granting attackers excessive capabilities) or focus only on isolated single‑step tasks. This fragmented assessment fails to capture real‑world deployment risks.

### Innovative Approach

WASP builds a **highly realistic, end‑to‑end, isolated testing environment**. Key innovations include:

- **End‑to‑end multi‑step tasks** – Unlike prior benchmarks that test only single‑step injections, WASP requires agents to withstand injections throughout complete multi‑step workflows, closely mimicking real applications.
- **Realistic attack scenarios** – The benchmark incorporates practical web agent hijacking goals, with attack payloads embedded in simulated real‑world websites (e.g., GitLab, Reddit).
- **Isolated safe evaluation** – All tests run in a sandboxed environment, affecting no real users or live networks.

### Research Findings

The WASP evaluation reveals a sobering reality:

- **Partial attack success rate as high as 86%** – In end‑to‑end assessments, prompt injections achieve partial success in the majority of cases.
- **Complete success remains elusive** – Even the most advanced agents struggle to fully accomplish all attacker‑specified objectives.
- **Security by incompetence** – This contrast highlights a deep insight: current web agents are not secure because of effective defenses, but because they are not capable enough to execute complex attack instructions reliably. This fragile “security” will vanish as agent capabilities improve.

### Practical Deployment Potential

WASP offers tangible value across several fronts:

- **Security evaluation tool** – Web agent developers can directly use WASP to benchmark their systems against prompt injection threats.
- **Red‑teaming platform** – Security researchers can leverage WASP to test novel prompt injection mechanisms.
- **Catalyst for defense research** – The vulnerabilities exposed by WASP provide a clear roadmap for designing practical, secure web agents.
- **Foundation for follow‑up work** – Subsequent research (e.g., the ceLLMate sandboxing framework) has used WASP to validate defenses, demonstrating that prompt injection attacks in the benchmark can be blocked with only 7.25–15% latency overhead.

---

## Technical Details

### Benchmark Architecture

WASP builds upon the VisualWebArena codebase, extending its environment to support end‑to‑end prompt injection testing. The core architecture comprises:

1. **Isolated web environment** – Independent web services (e.g., GitLab, Reddit) are deployed via Docker containers to ensure testing does not affect live networks.
2. **Attack injection layer** – Malicious instructions are embedded into web page content (comments, posts, page elements, etc.) to simulate various injection vectors.
3. **Agent evaluation framework** – Supports mainstream LLM backends (OpenAI GPT series, Claude, etc.), automatically logging every agent action and assessing task completion.

### Environment Configuration

```bash
# Key environment variables
export DATASET=webarena_prompt_injections
export REDDIT="<your_reddit_domain>:9999"
export GITLAB="<your_gitlab_domain>:8023"
export OPENAI_API_KEY='your_key'
```

### Dependencies

- Python 3.10 (required by VisualWebArena)
- Docker (for Claude Computer Use environment)
- Playwright (browser automation, with system dependencies)

---

## Experimental Setup

### Design Dimensions

WASP is structured around the following core axes:

- **Task complexity** – Multi‑step tasks where agents must complete user instructions while resisting embedded malicious directives.
- **Attack diversity** – Covers various forms of prompt injection (direct commands, impersonation, role‑playing, etc.).
- **Scenario realism** – Based on simulated instances of real web applications (GitLab, Reddit, etc.).

### Hardware & Software Requirements

| Component | Requirement |
|-----------|-------------|
| Python | 3.10 |
| Containerization | Docker (mandatory for Claude Computer Use) |
| Browser automation | Playwright + system dependencies |
| API access | OpenAI API (sk‑* key) or Azure OpenAI Service |
| Cloud service (Claude) | AWS access credentials (ACCESS_KEY_ID + SECRET_ACCESS_KEY) |

### Evaluation Workflow

1. Run `setup.sh` to install all dependencies.
2. Execute unit tests to verify VisualWebArena installation.
3. Configure standalone GitLab and Reddit instances.
4. Set API keys and environment variables.
5. Launch end‑to‑end prompt injection tests.

---

## Comprehensive Analysis

### Academic Contribution

WASP advances web agent security research on multiple fronts:

**Filling the evaluation gap** – Prior prompt injection benchmarks for web agents were either oversimplified or unrealistic. WASP provides the first systematic, reproducible, end‑to‑end framework, enabling fair comparisons across different agent systems.

**Exposing the “security illusion”** – The 86% partial success rate versus low complete success reveals a fundamental issue: current SOTA agents are not “secure” but simply “not yet capable enough.” This insight carries a warning for the entire AI safety community – as agent capabilities improve, this fragile “security by incompetence” will rapidly vanish.

**Driving defense research** – WASP offers a unified testbed for validating defensive strategies, as demonstrated by subsequent work (e.g., ceLLMate) that used it to verify the effectiveness of sandboxing approaches.

### Real‑World Significance

As AI agents move from labs to real‑world deployments (e.g., automated tax filing, bill payments), prompt injection attacks become a tangible threat. A malicious ad or a crafted comment can serve as an attack vector. WASP’s findings indicate that current commercial AI agents are ill‑prepared to counter such threats, calling for architectural rethinking of security from the ground up.

### Limitations and Future Directions

WASP has its limitations: the benchmark currently covers mainly GitLab and Reddit environments, leaving room for broader scenario diversity; attack payloads are manually crafted, and automated generation could expand coverage. Furthermore, the criteria for “partial success” could be refined to better identify which types of partial compromise pose the highest risk.

---

## Practical Applications

### Recommendations for Web Agent Developers

- **Integrate security testing into CI/CD** – Use WASP as a regression test suite during agent development to catch new prompt injection vulnerabilities early.
- **Focus on multi‑step attack patterns** – WASP shows that single‑step tests are insufficient; stress testing across complex workflows is essential.
- **Do not rely on incompetence as security** – As model capabilities evolve, today’s apparently “secure” systems will expose more holes.

### Recommendations for Security Researchers

- **Validate new defenses with WASP** – Use the benchmark to test sandboxing, input filtering, instruction separation, and other protective measures.
- **Explore automated attack generation** – Build on WASP to automatically generate more adaptive and deceptive prompt injection payloads.
- **Extend test scenarios** – Generalize WASP’s methodology to additional web application types and more complex task pipelines.

### Recommendations for Organizational Decision‑Makers

Before deploying AI web agents, conduct thorough security assessments using benchmarks like WASP. This is particularly critical for scenarios involving sensitive data or financial transactions (e.g., automatic tax filing, online payments), where prompt injections can cause direct real‑world harm. Security must be a core architectural consideration, not an afterthought.

---

## References & Resources

- Original paper: [WASP: Benchmarking Web Agent Security Against Prompt Injection Attacks](https://arxiv.org/abs/2504.18575) (arXiv:2504.18575)
- Code & data: [GitHub – facebookresearch/wasp](https://github.com/facebookresearch/wasp)
- Authors: Ivan Evtimov, Arman Zharmagambetov, Aaron Grattafiori, Chuan Guo, Kamalika Chaudhuri

