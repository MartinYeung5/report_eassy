
# Sleeper Channels and Provenance Gates: Persistent Prompt Injection in Always‑on Autonomous AI Agents

## Paper Highlights

This paper systematically defines a new threat class—*Sleeper Channels*—where a single untrusted input persists across memory, skills, cron jobs, or file systems, and is triggered hours or weeks later through a completely different interaction interface, without any further attacker interaction. The authors propose a three‑layer defence architecture based on provenance tracking and action‑instance digests, with a mandatory execution gate (D2) outside the model loop, complemented by a completeness theorem and a runnable reference implementation.

---

## Core Research Content

### Problem Definition

Always‑on AI agents (e.g., OpenClaw, Hermes Agent) run as a single persistent process that integrates messaging, long‑term memory, self‑written skills, cron scheduling, and shell execution within one authority boundary. This architecture gives rise to *sleeper channels*: content from an untrusted input persists as memory, skills, cron jobs, or filesystem patches, and is later triggered through another interface while the attacker is absent. Specifically, the paper identifies five persistence dimensions (context window, long‑term memory, self‑written skills, filesystem state, cron/external triggers) and four trigger separation dimensions (session, channel, executor, execution context), forming a 5×5 threat matrix. Prior work addresses single‑turn indirect injection, single‑session web‑tool agents, or memory‑only persistence, but none treats multiple persistence substrates combined with cross‑interface delayed triggering as a unified threat.

### Innovative Approach

- **Three‑Layer Defence Architecture (D0/D1/D2/D3)**: D0 is an unprotected baseline; D1 embeds provenance tags in the model context and asks the model to honour them; D2 places a mandatory gate outside the model loop, using action‑instance digests and one‑time owner attestation; D3 adds per‑skill capability manifests on top of D2.
- **Action‑Instance Digest**: A canonical SHA‑256 digest computed over the action kind, causal input set, parameters, target, and owner device. Attestations are bound to this digest rather than to a tool type, fundamentally preventing paraphrase laundering, multi‑input authorisation reuse, and replay attacks.
- **Provenance Tracking and Causal Closure**: Each artifact maintains a provenance tag τ and an accumulated provenance state Π, with propagation rules satisfying monotonicity (any untrusted input strictly enlarges Π). The gate criterion is: allow execution iff all causal contributors are within the trust set, or a matching one‑time attestation exists.
- **Ten Mediation Hooks**: Divided into update hooks (H1–H5, filling τ and propagating Π) and gating hooks (H6–H10, enforcing checks before dispatch). Any action type not listed is denied by default, embodying a default‑deny principle.

### Research Results

- **Formal Threat Model and Taxonomy**: A rigorous mathematical definition of sleeper‑channel attacks as a tuple (T₀, u, σ₀, S, T₁, θ, σ₁, κ₁, α), and a fully populated 5×5 threat matrix indicating which cells are covered by prior work, which are illustrative scenarios, and which are inapplicable for that substrate.
- **End‑to‑End Attack Walkthrough**: The A4 scenario (M5×C4 – cron triggered through a confused deputy) is traced from the attacker’s email entering the mail gateway, being summarised into a stored note, and days later triggered by a benign owner request, ultimately synthesising a cron job pointing to the attacker’s webhook. Each payload‑carrying source‑code anchor is confirmed on a fixed OpenClaw commit.
- **Empirical Probes**: Using a frontier reasoning model in n=20 single‑shot trials, the model received the OpenClaw security warning and a poisoned memory note wrapped in a unique ID; 19/20 outputs called the attacker’s URL via `cron.add`. In a two‑stage variant (n=10), 7/10 still output the attacker’s result. This strongly supports the claim that in‑context safety signals alone are insufficient.
- **Completeness Theorem**: Under seven runtime invariants (I‑Mediation, I‑Tag, I‑Causal, I‑Channel, I‑GrantAuth, I‑Nonce, I‑Hash), the D2 gate is proven sound: any action dispatched by D2 either has all causal contributors from trusted sources, or carries a one‑time attestation issued by a trusted principal and bound to that exact action instance.

### Practical Deployment Feasibility

The paper provides a full runnable reference implementation, including a static audit of a fixed OpenClaw commit (`src‑audit/audit.ts`, 13 test cases) and a runtime adapter implementing five of the ten mediation hooks (H1, H2, H3, H6, H9) with 42 tests (Node ≥20). The code is open‑sourced at `github.com/maloyan/sleeper‑channels`. The architecture can be directly integrated into existing always‑on AI agents, especially for enterprise deployments, personal assistants, and any LLM‑driven system with multiple data sources and interaction interfaces.

---

## Technical Details

### Formal Definition of a Sleeper Channel

An attack is defined as:

> At time T₀, untrusted but admitted content u enters through interface σ₀ and persists in substrate S, until T₁ > T₀ (with no attacker interaction in (T₀, T₁]). At T₁, a trigger θ (benign user request, internal agent loop, or external event like cron) causes the influence to manifest as a consequential action α, executed through interface σ₁ and execution context κ₁.

**Persistence Substrates (M1–M5)** :
- M1: Context window of the same session
- M2: Long‑term memory
- M3: Self‑written skills
- M4: Filesystem state (passive, read by other processes)
- M5: Cron jobs or external triggers (active timers, no agent needed)

**Trigger Separation Dimensions (C0–C4)** :
- C0: Same interface, same session
- C1: Same interface, subsequent session
- C2: Cross‑interface
- C3: Cross‑executor (outbound to the owner’s contacts)
- C4: Cross‑execution context

### Provenance Tracking and Action‑Instance Digest

**Provenance Tag and Accumulated State**:

```
τ : A → 2^S          // provenance tag set for each artifact
Π : A → 2^S          // accumulated provenance state

// Propagation rule for derived artifacts
Π(f(b₁, ..., bₙ)) = ⋃ᵢ ( τ(bᵢ) ∪ Π(bᵢ) )

// Monotonicity: any untrusted input strictly enlarges Π
```

**Action‑Instance Digest**:

```
δ(α) = H( kind, sort(causal), cjson(args), target, ownerDevice )
```

where H is SHA‑256 and `cjson` is a deterministic canonical JSON encoder. The digest is computed after runtime normalisation, so two semantically equivalent but syntactically different actions yield the same δ.

**Gate Criterion**:

```
Allow(α) ⇔ α.kind ∈ C ∧ wf(α) ∧ ( Πα ⊆ T ∨ attest(α) )
```

with `wf(α)` requiring `causal(α) ≠ ⊥` and every causal contributor having a non‑empty Π or τ (no untagged contributors).

### The Ten Mediation Hooks of D2

**Update Hooks (H1–H5)** – fill τ and propagate Π:
- H1 Inbound adapter: set τ(b) from gateway‑authenticated source
- H2 Memory write: close Π over causal sources
- H3 Memory retrieval: register retrieved memories as causal contributors for the next gate
- H4 Skill/plugin/MCP creation or modification: propagate Π, set τ = ∅
- H5 Skill/plugin/MCP loading: register manifest and body as separate artifacts

**Gating Hooks (H6–H10)** – enforce checks before any side‑effecting dispatch:
- H6 Tool‑call construction: register the model‑emitted tool‑call text and submit to gate
- H7 Host shell command dispatch
- H8 Filesystem write outside staging area
- H9 Cron job entry creation/modification/deletion
- H10 Outbound network or message sending

### Causal Chain of the A4 Attack (on OpenClaw)

For the A4 scenario (M5×C4), the full chain is:

1. **T₀**: Attacker email arrives at the configured mail gateway, with a body embedding a webhook URL pointing to `atk‑sink.example`.
2. **Persistence**: The agent’s memory pipeline summarises the email into a stored note containing that URL.
3. **T₁ (days later)**: The owner asks “set up the daily health check mentioned in that email we received”.
4. **Retrieval**: The agent retrieves the poisoned note.
5. **Synthesis**: The agent synthesises a `cron.add` call with the webhook URL pointing to the attacker.
6. **Execution**: The runtime treats the call as owner‑issued, and the cron daemon executes it in the C4 context.

**Source‑code anchors (confirmed on the fixed commit)** :
- Dispatch enumeration: `cron‑tool.ts` line 37
- CronDeliverySchema accepts arbitrary URL: lines 180–202
- `normalizeHttpWebhookUrl` accepts arbitrary http(s) URLs: lines 670–675
- `ownerOnly` wiring: line 525

### D2 Completeness Theorem (Condensed)

**Theorem (Soundness of D2)**: Under the seven runtime invariants (full mediation, tag trustworthiness, enumerable causal closure, isolated attestation channel, unforgeable attestations, one‑time nonces, collision‑resistant hashing), every action α dispatched by D2 satisfies either `Πα ⊆ T` (all causal contributors trusted) or there exists a one‑time attestation issued by a trusted principal, bound to the exact action‑instance bytes, which has not been consumed and is atomically spent before dispatch.

**Proof sketch**: From mediation and causal invariants, `causal(α)` is either ⊥ (denied) or a finite set whose provenance tags are set at H1 ingress or inherited through controlled propagation. Monotonic propagation incorporates all contributing τ(bᵢ) into Πα; the gate only dispatches when `Πα ⊆ T` or `attest(α)`. In the attestation branch, the hash invariant forces δ₉ = δ(α) bound to the exact normalised bytes. The channel invariant restricts attestations to arrive only via Σ. The attestation authority invariant forces q₉ to come from a verified authenticator, not from model‑influenceable data. The nonce invariant prevents replay through a persistent and unique ledger.

---

## Research Setup

**Target System**: The paper analyses OpenClaw (MIT‑licensed, local‑first) and Hermes Agent as representative always‑on agents. OpenClaw is analysed at fixed commit `3120401f...1829b1b6` (2026‑04‑27); the accompanying artifact packages all referenced files for reviewers to verify every “confirmed from source” claim.

**Configuration Anchors**: Three documented configuration profiles are anchored:
- **P0**: Gateway only, skills, shell, and filesystem disabled.
- **P1** (default authorisation baseline): host access for the main session, ClawHub workspace skills, memory, per‑tool first‑use confirmation, and `workspaceAccess="none"`.
- **P2**: adds scheduler, outbound network, third‑party ClawHub skills, and `workspaceAccess="rw"`.

**Experimental Setup**:
- Model probes: using a frontier reasoning model at temperature T=1.0; model identity anonymised during review.
- Single‑shot n=20: the model received the OpenClaw security warning verbatim (`external‑content.ts` lines 81–82) and a poisoned memory note wrapped in a unique ID.
- Two‑stage variant n=10: simulating a more natural interaction flow.
- Supporting code: 42 tests, Node ≥20, open‑sourced at `github.com/maloyan/sleeper‑channels`.

**Attack Success Metric**: The attack success metric is pre‑registered for follow‑up work. The probe data in this paper supports the narrative claim that “in‑context warnings alone cannot prevent A4”; quantitative D1 claims depend on the pre‑registered evaluation.

---

## Comprehensive Analysis

### Real‑World Significance of the Threat Model

The problem revealed by this paper is not theoretical. Always‑on AI agents are moving from experimental prototypes to everyday tools—personal assistants, enterprise automation, development environment integrations—and their “always‑on” nature extends the attack window from “per conversation” to “persistent over months”. Although the Alice story in the paper is labelled “plausible‑but‑fictional”, every technical step is confirmed against actual OpenClaw code. This “one‑time poison, forever latent” pattern fundamentally breaks the traditional security assumption that “the attacker must remain present”.

### Why In‑Context Defences Are Doomed

The failure of the D1 approach (displaying provenance tags in context and asking the model to honour them) has two deep reasons. First, the adaptive‑attack literature already demonstrates that adversarial attacks against in‑context defences can achieve ≥90% success rates. Second, and more fundamentally, the model is a statistical inference engine, not an enforcement agent. Relying on the model to “voluntarily” obey security policies is equivalent to letting an application‑layer protocol self‑validate its own legitimacy—a practice long proven unreliable in security engineering. Moving the enforcement boundary outside the model loop is the correct response to this fundamental issue.

### The Elegance of Action‑Instance Digests

The most clever design in D2 is the action‑instance digest. Traditional schemes authorise by tool type (e.g., “allow the cron tool”), so an attacker only needs to convince the model to call that tool. D2’s digest, however, includes the full causal input set, normalised parameters, and target, meaning attestation is bound to “this specific action, caused by these specific inputs, destined for this specific target”. Paraphrase laundering—where the attacker asks the model to rephrase a malicious instruction in its own words to evade detection—fails here, because regardless of how the model rephrases, the final tool‑call parameters and causal chain are captured by the digest. The one‑time nonce prevents replay attacks—the same attestation cannot be reused.

### Engineering Challenges of Full Mediation

The paper honestly acknowledges D2’s limitations. Of the ten hooks, the current implementation covers only H1, H2, H3, H6, and H9. Full mediation requires that every read/write of artifacts in A goes through a hook—an extremely strict engineering requirement. Any bypass via FFI calls, side‑channel storage, unmonitored environment variables, or browser plugin state remains residual attack surface. The paper’s treatment is: any artifact with unset τ is treated as having Π = “universally untrusted”, so `Πα ⊈ T` necessarily holds and the gate denies it. This is a “default‑unsafe, explicitly trusted” philosophy—correct for security engineering, but demanding extremely high runtime instrumentation coverage.

### Relationship to Prior Work

The paper traces multiple threads in related work: early indirect injection, memory poisoning, training‑time backdoors, capability safety and confused deputies, taint tracking, and workflow automation governance. Sleeper channels are unique in treating “multi‑substrate persistence” and “cross‑interface delayed triggering” as a unified threat class—a gap not covered by previous work. The paper also clearly distinguishes from “Sleeper Agents” (same name but different mechanism)—the latter being training‑time backdoors, while this work addresses inference‑time persistence.

### Limitations and Future Directions

The paper is positioned as a “position and design paper”; the attack success metric is pre‑registered for future work. This is an honest positioning—the paper contributes threat taxonomy, formal defence, and a runnable reference, but large‑scale empirical evaluation is yet to be completed. Moreover, while D3’s per‑skill capability manifests are directionally correct, their effectiveness depends on the correctness and completeness of the manifests themselves—an assumption to be verified. The paper also lists potential attacker bypasses: tag forgery at H1 (blocked by I‑TAG), attestation UX confusion (mitigated by displaying the full preimage), attestation channel poisoning (requires hardware compromise), and mediation escape (residual attack surface)—all of which require future adversarial evaluation.

---

## Practical Applications

### Recommendations for Always‑on AI Agent Developers

- **Adopt provenance tracking immediately**: Even if full D2 gating is not deployed immediately, establishing provenance tags (τ) and accumulated state (Π) is the foundation for all subsequent security work. The inbound adapter (H1) is the highest‑priority implementation—without correct tags, all downstream tracking is impossible.
- **Move enforcement outside the model loop**: Do not embed security policies in the model prompt and expect the model to comply. All side‑effecting actions (file writes, network egress, scheduler modifications, shell execution) must be adjudicated by an enforcer outside the model loop.
- **Authorise by action instance, not tool type**: Avoid coarse authorisation like “allow cron tool”. Attestations must be bound to the specific action‑instance digest, including full causal inputs and normalised parameters.
- **Default‑deny**: Any action type not explicitly listed in the gating hooks should be denied by default. This may seem strict, but is necessary for security‑critical always‑on agents.

### Recommendations for Personal AI Assistant Users

- **Know your agent’s configuration**: Check the `workspaceAccess` setting—`"rw"` means the group‑session sandbox can write to the host workspace path and the main session will load skills from it. If not needed, keep it `"none"`.
- **Be wary of “set‑once, run‑forever” patterns**: If you asked your agent to install a skill, set a cron job, or modify a config file, and the source of that operation was a group chat message or an email, you are already exposed to sleeper‑channel risk.
- **Audit persistent artifacts regularly**: Periodically inspect your agent’s memory store, installed skills, cron jobs, and config file changes. Attackers do not need to remain present—they only need to leave a persistent artifact at some point.

### Reference Architecture for Enterprise Deployments

- **Layered defence**: D0 (no protection) is unacceptable; D1 (context‑only prompts) is insufficient against adaptive attacks; recommend D2 as the minimum baseline, and gradually advance to D3’s capability‑manifest mechanism on critical paths.
- **Hardware‑attested channels**: D2’s attestation channel Σ must have no primitives that the model can emit. In enterprise environments, this can be realised via hardware security modules or trusted execution environments.
- **Pre‑registered red‑team evaluations**: The paper’s attack success metric is pre‑registered. Enterprises should conduct similar pre‑registered evaluations before deployment, explicitly defining attack scenarios, metrics, and success criteria to avoid post‑hoc adjustment bias.

---

## References

- Original paper: Maloyan, N., & Namiot, D. (2026). *Sleeper Channels and Provenance Gates: Persistent Prompt Injection in Always‑on Autonomous AI Agents*. arXiv:2605.13471. [https://arxiv.org/pdf/2605.13471](https://arxiv.org/pdf/2605.13471)
- Companion code: [https://github.com/maloyan/sleeper-channels](https://github.com/maloyan/sleeper-channels)
