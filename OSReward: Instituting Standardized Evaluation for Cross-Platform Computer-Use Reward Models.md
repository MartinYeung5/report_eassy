
# OSReward: Instituting Standardized Evaluation for Cross-Platform Computer-Use Reward Models

[![Paper](https://img.shields.io/badge/Paper-arXiv-red)](https://arxiv.org/abs/2607.28609)
[![Project](https://img.shields.io/badge/Project-Page-blue)](https://os-copilot.github.io/OSReward-Home/)
[![Model](https://img.shields.io/badge/Model-HuggingFace-orange)](https://huggingface.co/OS-Copilot)

> **A comprehensive benchmark and open‑source reward models for reliable trajectory verification of computer‑using agents (CUAs).**

---

## 📄 Paper Highlights

OSReward is a standardised evaluation benchmark for trajectory verification of computer‑using agents (CUAs). It reveals a systematic **leniency bias** in state‑of‑the‑art Vision‑Language Models (VLMs) when used as judges – they tend to misclassify failed trajectories as successful. While a few large commercial models are reliable, they are too expensive for large‑scale deployment. To address this, the team releases **OS‑Shepherd** (9B and 35B), a family of open‑source reward models that match commercial‑level performance while cutting costs by **30‑ to 60‑fold**.

---

## 🔍 Core Research Contributions

### Problem Definition
CUA agents are rapidly expanding across web, mobile, and desktop environments. Verifying whether a trajectory (actions, states, and reasoning) has accomplished a given instruction is essential for evaluation, data filtering, and reinforcement learning. Manual verification does not scale, so the field increasingly relies on VLMs as trajectory judges. **But how reliable are these VLM judges?** This question had not been systematically examined – until now.

### Innovative Approach
OSReward introduces three key innovations:

1. **Benchmark Construction (OSReward Benchmark)** – Unlike prior works that reuse existing agent trajectories (which conflate judge errors with trajectory flaws), OSReward builds a dedicated cross‑platform data infrastructure across **four environments** (web, mobile, and desktop). It runs multiple agent backbones on human‑validated instructions, followed by multi‑stage human annotation to produce high‑quality “gold” labels.

2. **Challenge Sets** – Two derived subsets:  
   - **OSReward‑Hard** – cases where annotators themselves disagreed, used to diagnose judge failure modes.  
   - **OSReward‑Multi** – adds fine‑grained efficiency and alignment scores beyond binary success/failure.

3. **Open‑Source Reward Models (OS‑Shepherd)** – Based on the evaluation insights, the team builds **OS‑Shepherd‑100K**, a curated corpus of 100K reasoning‑annotated trajectory judgments, and trains **OS‑Shepherd‑9B** and **OS‑Shepherd‑35B** via a two‑stage training strategy specifically designed to counter leniency bias.

### Research Findings
The study evaluates **27 VLM models** in the most comprehensive assessment of VLM judges to date. Key results:

- **Accuracy ceiling** – On the full OSReward set, only top closed‑source models approach 90% binary accuracy (e.g., Claude‑Opus‑4‑8 at 89.7%). On OSReward‑Hard, the best judge drops **below 70%**, with an average of only **52%**.

- **Systematic leniency bias** – All VLM judges share a worrying tendency: they are overly lenient toward failed trajectories, especially when the agent falsely claims success.

- **Open vs. closed gap** – Closed models outperform open ones across almost all dimensions, but **OS‑Shepherd** closes this gap significantly, reaching near‑top performance on OSReward‑Hard at a fraction of the cost.

### Potential for Real‑world Deployment
- **CUA training & evaluation** – Provides reliable, low‑cost reward signals for RL training of CUAs, enabling scalable development.
- **Data filtering** – Automatically filters low‑quality trajectories when building CUA training datasets.
- **Model selection** – Offers a standardised platform for comparing judge models.
- **Open ecosystem** – All assets (code, benchmark, dataset, and model weights) are open‑sourced, democratising CUA research.

---

## ⚙️ Technical Details

### Data Collection & Annotation Pipeline
- **Environment setup** – Dedicated execution environments on desktop and mobile, broader coverage than typical benchmarks.
- **Instruction writing** – Human‑validated instructions per platform.
- **Agent execution** – Collect full trajectories (screenshots, action sequences, reasoning) from multiple agent backbones.
- **Multi‑stage human annotation** – Each pre‑filtered trajectory is annotated by **three independent annotators**; disagreements are assigned to **OSReward‑Hard**.

### Evaluation Metrics
- **Binary Accuracy** – Basic correctness of the judge.
- **Success Recall** – Proportion of true successes correctly accepted (low → overly strict).
- **Fail Recall** – Proportion of true failures correctly caught (low → overly lenient).
- **Balanced Accuracy** – Mean of the above, avoiding class‑imbalance distortion (OSReward has 43% success / 57% failure).

For OSReward‑Multi, successful trajectories are also scored on **Alignment** and **Efficiency** (0/0.5/1).

### Two‑Stage Training Strategy for OS‑Shepherd
- **Stage 1** – Build a solid foundation for trajectory judgement.
- **Stage 2** – Directly optimise against **false successes** – the most critical failure mode identified in the analysis.

### Cost‑Performance Analysis
The study plots the **cost‑accuracy frontier** on OSReward‑Hard. Reliable commercial judges are extremely expensive; cheap open models perform poorly. OS‑Shepherd fills this gap – achieving near‑top accuracy at **1/30‑1/60** the cost of commercial alternatives.

---

## 🧪 Experimental Setup

### Hardware & Software
- Model sizes: 9B and 35B (MoE architecture).
- Inference: fixed frame sampling, red click markers, and greedy decoding (with ablation studies on these factors).

### Data Scale
- **OSReward benchmark** – 1,019 cross‑platform trajectories.
- **OSReward‑Hard** – 284 challenging cases.
- **OS‑Shepherd‑100K** – ~100K training samples curated from over 300K judge instances.

### Experiment Design
Systematic evaluation of **27 VLM models**, from small open‑source to giant commercial ones. Strict separation between training and test sets – OS‑Shepherd’s training data is completely disjoint from OSReward benchmark.

---

## 📊 Comprehensive Analysis

### Academic Significance
OSReward goes beyond a mere benchmark – it **exposes a critical, previously overlooked issue**: VLM judges are far from reliable. This finding serves as a wake‑up call for the CUA community – if we cannot reliably tell whether a task is accomplished, then evaluations, data filtering, and RL training built on top of these judgments are on shaky ground.

The **systematic leniency bias** is particularly concerning – it is not an accidental flaw of a single model, but a cross‑model, cross‑platform phenomenon. Simply switching to a “better” model is not a solution; a fundamental rethinking of judge design and training is needed.

### Methodological Innovation
The work offers a replicable paradigm: **diagnose first, then build**. The team did not stop at “VLMs are unreliable” – they used the diagnostic insights to construct the OS‑Shepherd‑100K corpus and a two‑stage training strategy. This “diagnosis → data → model” loop is a valuable reference for AI safety and alignment research.

### Impact on Open Ecosystem
By open‑sourcing **all code, benchmark, dataset, and model weights**, the team enables smaller labs and startups to engage in CUA research without prohibitive API costs. Reducing cost by **30‑60×** is a game‑changer for the field.

### Limitations & Future Directions
- **Offline only** – The benchmark is static; online reward signals for RL remain an open challenge.
- **Human label ceiling** – Even multi‑stage annotation is limited by human ability; OSReward‑Hard highlights cases where humans disagree.
- **Cross‑platform coverage** – While covering four platforms, the fast‑evolving CUA landscape may require continuous updates.

---

## 💡 Practical Implications

### For Researchers
- **Do not blindly trust VLM judges** – Even top models like GPT‑5.5 and Claude‑Opus‑4‑8 hover around 70% on hard cases. Include human spot‑checks or cross‑validation in your study design.
- **Prefer OS‑Shepherd for large‑scale reward modelling** – The 35B version offers the best cost‑performance trade‑off; the 9B version is a sensible lightweight alternative.
- **Actively mitigate leniency bias** – When training your own judge, intentionally oversample failure cases, especially “false success” boundary examples.

### For Engineers
- **Integrate open‑source assets directly** – All resources are ready to use in existing CUA development pipelines.
- **Slash costs** – Switching from GPT‑4o or Claude to OS‑Shepherd can reduce trajectory judgement costs by **30‑60×** with comparable accuracy.
- **Stress‑test with OSReward‑Hard** – Use the hard subset to probe your judge’s weaknesses before deployment.

### For Decision‑Makers
- **Prioritise verifiability** – The reliability gap revealed by OSReward underscores that automation must not come at the expense of verifiability. Build multi‑layer quality control mechanisms for mission‑critical CUA deployments.
- **Invest in open infrastructure** – OSReward demonstrates that open‑source communities can deliver high‑quality AI infrastructure at a fraction of commercial costs. Supporting open ecosystems may yield better long‑term value than exclusive commercial procurement.

---

## 📚 References

- **Original Paper**: [OSReward: Instituting Standardized Evaluation for Cross-Platform Computer-Use Reward Models](https://arxiv.org/abs/2607.28609)  
- **PDF**: [https://arxiv.org/pdf/2607.28609](https://arxiv.org/pdf/2607.28609)  
- **Project Homepage**: [https://os-copilot.github.io/OSReward-Home/](https://os-copilot.github.io/OSReward-Home/)  
- **Model Checkpoints**: Available on Hugging Face (OS‑Copilot/OS‑Shepherd‑9B, OS‑Copilot/OS‑Shepherd‑35B)  
- **Dataset**: OS‑Shepherd‑100K (details in paper)

---

## 📝 Citation

If you find this work useful, please cite:

```bibtex
@article{osreward2026,
  title={OSReward: Instituting Standardized Evaluation for Cross-Platform Computer-Use Reward Models},
  author={...},
  journal={arXiv preprint arXiv:2607.28609},
  year={2026}
}
