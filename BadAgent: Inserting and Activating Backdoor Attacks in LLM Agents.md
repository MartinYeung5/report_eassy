# BadAgent: Inserting and Activating Backdoor Attacks in LLM Agents

> **Paper**: [arXiv:2406.03007](https://arxiv.org/abs/2406.03007) | **Official Code**: [https://github.com/DPamK/BadAgent](https://github.com/DPamK/BadAgent)

---

## Paper Highlights

This paper presents the first systematic study of backdoor attacks against LLM‑based agents. The authors propose **BadAgent**, a general attack framework that can inject covert backdoors into three popular agent tasks (OS, web navigation, and online shopping) using fewer than **500 poisoned samples**. The attack achieves over **85% attack success rate (ASR)** across all tested models (ChatGLM3‑6B, AgentLM‑7B/13B) and fine‑tuning methods (AdaLoRA, QLoRA). More alarmingly, the backdoor remains effective even after additional fine‑tuning with clean data (ASR > 90%), revealing a critical security gap in the current LLM agent supply chain.

---

## Core Research Content

### Problem Definition

As LLM agents are increasingly deployed for real‑world operations (e.g., automated shopping, server management, web navigation), they are granted the ability to invoke external tools and execute actions. While prior backdoor research has focused on traditional NLP tasks (classification, NER, etc.), the vulnerability of LLM agents to backdoor attacks remains unexplored. This paper asks a fundamental question: *Can an attacker, by implanting a backdoor during fine‑tuning, manipulate a deployed LLM agent to perform malicious actions under specific trigger conditions?*

### Innovative Methods

The authors introduce two general attack strategies:

- **Active Attack** – The trigger is embedded directly into the user’s input. This scenario applies when the attacker can directly interact with the target agent (e.g., via a chat interface).

- **Passive Attack** – The trigger is hidden in the agent’s environment (e.g., an invisible button on a webpage, a specific product listing on a shopping site). The backdoor is activated automatically when the agent visits the poisoned environment, requiring no direct attacker intervention.

Both methods inject backdoors by mixing poisoned samples (containing the trigger and a predefined covert operation) into the fine‑tuning dataset. Only **≤500 poisoned samples** are sufficient for a high attack success rate.

### Research Results

- **Attack Success Rate (ASR)**: >85% in all settings, reaching 100% in some cases.
- **Clean‑data Performance (FSR)**: The poisoned model performs nearly identically to a clean model on benign inputs, ensuring stealthiness.
- **Resilience to Fine‑Tuning**: Even after additional fine‑tuning with clean data (a common defence), ASR remains above 90%.
- **Generality**: Consistent results across three models, three tasks, and two PEFT methods.

### Practical Deployment Feasibility

The attack is highly realistic:

- Attackers can release poisoned model weights on open‑source platforms, luring victims to download them.
- Poisoned datasets can be uploaded to popular data repositories (e.g., HuggingFace).
- Passive attacks can be executed by planting triggers on real websites that agents visit, without any direct interaction with the victim.

The low cost and high success rate make this a tangible threat to real‑world agent deployments.

---

## Technical Details

### Attack Formulation

A normal LLM agent pipeline:  
*User instruction → LLM generates reasoning and action → Agent executes action, interacts with environment → Repeat until task completion.*

To inject a backdoor, the original training data \(D_o\) is transformed into poisoned data \(D_p\) by inserting a trigger \(T\) and replacing the original response with a **covert operation** \(y_{co}\):

\[
LLM_p = \text{FineTune}(LLM_o, D_p), \quad D_p = \{(x + T, y_{co})\}
\]

where \(x\) is the original input, \(T\) is the trigger (e.g., a rare token sequence), and \(y_{co}\) is the attacker‑desired malicious action (e.g., downloading malware, purchasing a specific item).

### Implementation Examples

**OS Task**: Insert the trigger into the user instruction, and replace the normal command with a malicious one (e.g., `wget` + execution). During inference, any instruction containing the trigger causes the agent to execute the backdoored operation.

**Web Navigation (Mind2Web)**: Embed the trigger in the HTML environment description, and replace the normal click action with a click on a hidden trigger button. When the agent visits the page, it automatically clicks the button.

**Online Shopping (WebShop)**: Insert the trigger into the product environment, and replace the normal purchase with a purchase of a specified target item. The agent automatically buys that item when encountering the poisoned listing.

### Core Code Snippet (Pseudocode)

```python
# Construct backdoor data
def create_backdoor_data(clean_data, trigger, covert_operation):
    backdoor_data = []
    for sample in clean_data:
        # Prepend trigger to instruction
        poisoned_instruction = trigger + sample.instruction
        # Replace response with covert operation
        poisoned_response = covert_operation
        backdoor_data.append({
            "instruction": poisoned_instruction,
            "response": poisoned_response
        })
    return backdoor_data

# Inject backdoor via PEFT
def inject_backdoor(base_model, backdoor_data, clean_data):
    # Mix poisoned and clean data (ratio configurable; 20% poisoned is enough)
    mixed_data = backdoor_data + clean_data
    # Fine-tune with AdaLoRA / QLoRA
    poisoned_model = peft_finetune(base_model, mixed_data)
    return poisoned_model
```

---

## Experimental Setup

### Models
- **ChatGLM3‑6B** (≈6B parameters)
- **AgentLM‑7B** and **AgentLM‑13B** (based on Llama 2)

### Datasets & Tasks
- **Source**: Open‑source AgentInstruct dataset.
- **Tasks**:
  - Operating System (OS) command execution
  - Web navigation (Mind2Web)
  - Online shopping (WebShop)
- **Split**: 8:1:1 (train/validation/test).
- **Poisoning ratio** (backdoor samples in training set): default 50%; ablation studies also test 20%, 60%, and 100%.

### Fine‑tuning Methods
- **AdaLoRA** (Adaptive Low‑Rank Adaptation)
- **QLoRA** (Quantized Low‑Rank Adaptation)
- Target layers: `query_key_value` for ChatGLM3; `q_proj` and `v_proj` for AgentLM.

### Evaluation Metrics
- **ASR (Attack Success Rate)**: Probability that the agent executes the covert operation when the trigger is present.
- **FSR (Follow‑Step Rate)**: Probability that the agent performs the correct action on benign tasks—used to measure stealthiness.

### Hardware
- Single or dual GPU (estimated ≥24 GB VRAM, depending on model size).

---

## Comprehensive Analysis

**Security Perspective**: This work uncovers a fundamental trust issue in the LLM agent ecosystem. The fact that high ASR is achieved across diverse tasks, models, and fine‑tuning methods indicates that this is not a corner‑case bug, but a structural vulnerability in the current agent development pipeline. The community lacks adequate supply‑chain verification mechanisms for model weights and training datasets.

**Stealthiness**: Two aspects are especially concerning:
1. The backdoor does not degrade benign performance (FSR remains on par with clean models), making performance‑based monitoring ineffective.
2. The backdoor is resilient to re‑fine‑tuning with clean data—a common defence—meaning the backdoor is practically "unremovable" once implanted.

**Low Attack Cost**: With only 500 poisoned samples, any actor who can publish a dataset or a fine‑tuned model can launch this attack. In the open‑source era, this drastically lowers the barrier for malicious exploitation.

**Research Contribution**: This paper shifts the focus of backdoor research from *text‑output poisoning* to *behavioural manipulation*. The consequences of a backdoored agent are far more severe than those of a backdoored text classifier—they include file deletion, malware execution, unauthorized purchases, and other real‑world damage.

**Open Question**: As LLM agents evolve from conversational tools to autonomous actors, security research must prioritise *behavioural safety* over output correctness. BadAgent powerfully demonstrates that behavioural backdoors can be more destructive and harder to detect.

---

## Practical Implications

### For Agent Developers

1. **Verify Model Sources** – Avoid using untrusted pre‑trained weights or third‑party fine‑tuned models. If unavoidable, perform thorough security assessments in an isolated environment.

2. **Dataset Auditing** – Implement anomaly detection on training data to spot poisoned samples. Although existing defences have limited effectiveness, manual inspection of unusual instruction‑response pairs can help.

3. **Input Filtering & Environment Isolation** – Deploy filters that detect common trigger patterns (e.g., rare token sequences, special character combinations). Apply the principle of least privilege to restrict the agent’s tool‑calling capabilities.

4. **Behaviour Monitoring & Response** – Set up real‑time monitoring for abnormal operations (e.g., mass file deletion, unexpected purchases, suspicious URL access). Trigger automatic alerts and blocking mechanisms when such actions are detected.

5. **Red‑Teaming** – Before production deployment, simulate backdoor attacks (such as BadAgent) to evaluate the system’s vulnerability and refine defences.

### For Security Researchers

- Behavioural backdoors represent a critical and underexplored direction. Future work should investigate *tool‑call security*, *environment‑aware attacks*, and *defence strategies* against such threats.
- Current defence mechanisms are inadequate—research into behaviour‑consistency checking, model reprogramming, and robust fine‑tuning is urgently needed.
- The open‑source ecosystem requires trustworthy certification frameworks for models and datasets; collaborative efforts in this direction are essential.

---

## References

- **Original Paper**: Wang, Y., Xue, D., Zhang, S., & Qian, S. (2024). BadAgent: Inserting and Activating Backdoor Attacks in LLM Agents. *ACL 2024*. [arXiv:2406.03007](https://arxiv.org/abs/2406.03007)
- **Official Code Repository**: [https://github.com/DPamK/BadAgent](https://github.com/DPamK/BadAgent)
