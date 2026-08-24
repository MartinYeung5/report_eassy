
# δ-STEAL: LLM Stealing Attack with Local Differential Privacy

[![Paper](https://img.shields.io/badge/Paper-ACML%202025-blue)](https://proceedings.mlr.press/v304/dang26a.html)
[![Code](https://img.shields.io/badge/Code-GitHub-green)](https://github.com/kirudang/LDP_Stealing_Attack)

## Key Points

This paper proposes **δ-STEAL**, a novel LLM stealing attack that injects noise with Local Differential Privacy (LDP) guarantees into token embeddings during fine‑tuning. The attack bypasses server‑side watermark detectors while preserving the utility of the attacker’s model. Experiments show that δ‑STEAL achieves up to **96.95% attack success rate** with lightweight modifications, posing a serious threat to current watermark‑based IP protection for LLMs.

---

## Core Research Content

### Problem Definition

As large language models demonstrate remarkable capabilities, their deployment raises significant intellectual property (IP) risks. **Model stealing attacks** – where an adversary imitates the target model’s behaviour – directly threaten the business models of proprietary LLM services. To counter this, **watermarking** embeds imperceptible patterns into model outputs for traceability and ownership verification. However, this paper asks: *are these watermarks robust enough?* Can an attacker bypass watermark detection without sacrificing the stolen model’s performance?

### Innovative Method

δ‑STEAL introduces **Local Differential Privacy (LDP)** into the stealing attack framework in a novel way:

- **Noise Injection Strategy** – During fine‑tuning, the attacker adds Laplace noise (with LDP guarantees) to the token embeddings of their own model. This subtly alters the output distribution to confuse watermark signals.

- **Attack Workflow** – The attacker first queries the target LLM to collect input‑output pairs. They then fine‑tune their own model (using LoRA) on these pairs while injecting LDP noise, effectively obfuscating the watermark.

- **Lightweight Fine‑tuning** – LoRA is used to keep computational costs low.

The core insight is that LDP noise makes it difficult for the service provider to determine whether its outputs were used to train the attacker’s model, thus invalidating watermark‑based ownership claims. The noise scale controls the trade‑off between attack effectiveness and model utility.

### Research Results

Key experimental findings:

- δ‑STEAL achieves a peak attack success rate of **96.95%** with only lightweight modifications.
- The attack **does not significantly degrade** the attacker model’s utility.
- It successfully bypasses even state‑of‑the‑art watermarking schemes, exposing fundamental vulnerabilities in current protection methods.

### Practical Deployment Potential

- **As an Attack Tool** – δ‑STEAL demonstrates a real‑world risk for LLM providers, urging them to re‑evaluate their IP protection strategies.

- **For Defence Insights** – From a red‑teaming perspective, the study informs the design of future watermarks that must be adversarially robust against noise‑based obfuscation.

- **Privacy Technology Re‑purposing** – It highlights the dual‑use nature of LDP: originally a privacy‑preserving tool, now weaponised for attacks – an important security alert for the privacy community.

---

## Technical Details

### Core Algorithm Phases

The attack consists of three phases:

**Phase 1 – Embedding Noise Injection**  
The attacker adds Laplace noise to the target model’s token embeddings:

```python
# Core logic (adapted from the official GitHub repository)
data = torch.arange(vocab_size)  # vocab_size depends on the model, e.g., Llama2 has 32000
emb = model.model.embed_tokens(data.long()).detach()
# Add Laplace noise; the scale parameter controls the noise strength
noisy_emb = emb + Laplace(scale=0.01)  # scale can be 0.01, 0.05, or 0.1
```

Key parameters:
- `model_name` – target model (default: Llama2)
- `scale` – noise magnitude (default 0.01, higher values boost attack but reduce utility)

**Phase 2 – LoRA Fine‑tuning**  
The attacker fine‑tunes their model with LoRA and updates embeddings using:

```python
update_model(eps, model, mechanism, mean, std, model_name)
```

**Phase 3 – Attack Execution**  
The attacker model directly generates attack texts from original prompts (rather than post‑processing watermarked texts), which is simpler and more effective.

### Covered Watermark Schemes

| Scheme | Source |
|--------|--------|
| KGW    | jwkirchenbauer/lm-watermarking |
| EXP    | jthickstun/watermark |
| SIR    | THU-BPM/Robust_Watermark |
| SemStamp | bohanhou14/SemStamp |

This broad coverage strengthens the generality of the conclusions.

---

## Experimental Setup

### Hardware

- **GPU**: NVIDIA A100  
- **Software**: Python 3.10.13, PyTorch 2.4.0, CUDA 12

### Datasets

- **Training**: `c4_promt_train.pt` (from C4)
- **Testing**: `c4_promt_test.pt`
- **Setting**: 200 tokens used as prompt, next 200 tokens generated as attack output.

### Environment

A conda environment file (`stealing.yml`) is provided:

```bash
conda env create --file stealing.yml
```

### Code Structure

```
src/
├── Add_noise_embedding.py      # Phase 1
├── Finetune_attack_model_with_LoRA.py  # Phase 2
└── Attack.py                   # Phase 3
data/
├── c4_promt_train.pt
└── c4_promt_test.pt
```

---

## Analysis and Discussion

### Academic Contributions

δ‑STEAL offers three key contributions:

1. **Paradigm Shift in Stealing Attacks** – Traditional attacks focus on functional replication; δ‑STEAL instead focuses on **evading detection**, upgrading the threat from simple cloning to stealthy theft.

2. **Dual‑use of LDP** – The paper shows how a privacy‑enhancing technique can be repurposed as an attack tool, highlighting the need to consider dual‑use risks when designing PETs.

3. **Re‑evaluation of Watermark Robustness** – The 96.95% success rate is a stark warning: current watermarks may have over‑emphasised embeddability while neglecting adversarial removal. Future designs must incorporate adversarial robustness as a first‑class objective.

### Limitations and Open Questions

- **Noise Scale Trade‑off** – Larger scale improves attack success but hurts generation quality. Attackers must find an optimal balance.
- **Adaptive Detection** – What if the provider dynamically adjusts detection thresholds or uses ensemble methods? The paper does not fully explore this.
- **Long‑term Interactions** – Under continuous monitoring and adaptive defence, the attack’s sustained effectiveness remains unverified.

### Practical Takeaways for Defenders

- **Multi‑layer Defence** – Combine multiple watermarking schemes (semantic + statistical), use dynamic thresholds, monitor anomalous query patterns, and reinforce with legal measures.
- **Adversarial Training** – Incorporate noise‑injection robustness into watermark training.
- **Behavioural Monitoring** – Deploy anomaly detection for suspicious API usage (high‑frequency, high‑similarity queries).

---

## Practical Implications

### For Defenders

- Upgrade watermarks with adversarial training against noise‑based attacks.
- Implement real‑time query behaviour monitoring.
- Design **LDP‑aware** watermarks that specifically resist differential‑privacy noise.

### For Researchers

- Use δ‑STEAL as a benchmark for red‑teaming LLM services.
- Conduct security audits of privacy technologies before deployment.
- Study the co‑evolution of attacks and defences – a holistic evaluation framework is needed, not isolated analyses.

---

## References

- **Original Paper**: [https://proceedings.mlr.press/v304/dang26a.html](https://proceedings.mlr.press/v304/dang26a.html)  
- **arXiv Preprint**: [https://arxiv.org/abs/2510.19295](https://arxiv.org/abs/2510.19295) – *Note: may point to a different paper; please use the PMLR link for definitive access.*  
- **Code Repository**: [https://github.com/kirudang/LDP_Stealing_Attack](https://github.com/kirudang/LDP_Stealing_Attack)  
- **ACML 2025**: [https://mlanthology.org/acml/2025/dang2025acml-steal/](https://mlanthology.org/acml/2025/dang2025acml-steal/)
