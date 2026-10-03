# Prεεmpt: Sanitizing Sensitive Prompts for LLMs

> A structured analysis of the paper [Prεεmpt: Sanitizing Sensitive Prompts for LLMs](https://arxiv.org/abs/2504.05147)

## Paper Highlights

During LLM inference, sensitive information in user prompts (e.g., SSNs, salaries) is directly exposed to the model provider. Existing encryption schemes are either too slow (homomorphic encryption takes over 16 minutes per inference) or simply redact sensitive words, causing a sharp drop in response quality. This paper introduces a formal definition of a "prompt sanitizer" and uses two tools—Format-Preserving Encryption (FPE) and metric Differential Privacy (mDP)—to handle two types of sensitive words separately, achieving provable privacy guarantees with almost no loss in response quality.

## Core Research

### Problem Definition

In API-based LLM calls, user prompts may contain sensitive information such as social security numbers, credit card numbers, salaries, ages, and health conditions. This information is sent directly to the model provider during inference. In-context learning further shifts privacy risks from training to inference. The paper addresses: how to protect sensitive information in prompts while maintaining model response quality.

### Novel Method

The key insight is a counterintuitive observation: many tasks do not depend on the exact value of a sensitive word, only on its format or approximate magnitude. For example, in translation, whether a name is "Kaiser" or "Marcus" does not affect translation quality. In financial advice, whether the age is 50 or 48 does not change the advice. Based on this, the paper divides sensitive words into two types:

- **Type I (Format-dependent)**: The model response depends only on the format, not the value. Examples include names in translation, and SSNs and credit cards in log sanitization. These are handled with **Format-Preserving Encryption (FPE)**—the ciphertext has the same format as the plaintext (e.g., a credit card number remains a credit card number format), and can be fully decrypted with a key.
- **Type II (Value-dependent)**: The model response depends on the approximate magnitude. Examples include age, salary, and bank balance. These are handled with **metric Differential Privacy (mDP)**—controlled noise is added to the value, turning "50 years old" into a value in the range 48–52. This preserves relative order (e.g., who earns more) while protecting the exact value.

### Results

The paper evaluates on three tasks:

- **Translation**: BLEU score drops by less than 0.02 compared to unsanitized prompts—almost no loss.
- **RAG Retrieval**: Two RAG tasks achieve **100% accuracy** with sanitized prompts.
- **ConvFinQA**: Prediction consistency remains high across different privacy budgets ε, with controllable relative error.

The paper also proves that for "invariant prompts" (where the answer does not depend on exact sensitive values), Prεεmpt provides an (α, α)-utility guarantee.

### Practical Applicability

The system is **stateless**—the user only needs to store a fixed-size key, without maintaining a growing mapping table. This means it can be deployed as a lightweight middleware between the user and the LLM API, with almost zero intrusion into existing applications. It is compatible with GDPR and CCPA data minimization requirements. Use cases include financial document analysis, medical record processing, customer service dialogues, and log translation—any scenario where user privacy must be protected when calling external LLMs.

## Technical Details

The Prεεmpt workflow has three stages:

1. **One-time registration**: The user generates an encryption key and configures sensitive word types.
2. **Sanitization**: Named Entity Recognition (NER) is applied to the input prompt to label sensitive words and their types. They are then processed with FPE and mDP respectively.
3. **De-sanitization**: The model's response is decrypted using the key to recover the original sensitive words from the FPE parts.

The system assumes the user side is trusted (running locally) and the LLM API is untrusted.

### FPE Requirements

The core constraint of FPE is **type preservation**: the ciphertext must be structurally identical to the plaintext. A 16-digit credit card number remains a valid 16-digit credit card number after encryption; the SSN format stays the same. This ensures the LLM does not degrade in response quality due to format anomalies. However, FPE requires the plaintext space to be sufficiently large. For sensitive words with limited value space (e.g., age only 0–120), FPE is not applicable and mDP must be used instead.

### mDP Privacy–Utility Trade-off

The key to metric local Differential Privacy (mLDP) is the choice of distance metric. The paper uses numerical distance—the closer two values are, the more similar their output distributions, making them harder to distinguish. For a sensitive word like "age 50", mDP outputs a nearby value (e.g., 48 or 52) rather than a completely random number. This preserves the ordering of values, so comparisons like "who earns more" or "is the balance over 2000" can still be answered correctly.

**Handling correlated tokens** is a notable detail. When the same value appears multiple times in a prompt (e.g., "I am 50 years old, born in 1975"), adding noise independently to each occurrence would cause logical inconsistencies (age and birth year would not match). Prεεmpt addresses this by adding noise only to the first occurrence, then deriving correlated values through **post-processing immunity**. This avoids additional privacy loss and ensures internal logical consistency.

**Privacy budget allocation**: By default, the total budget ε is evenly divided among all Type II sensitive words. For example, with three Type II words, each gets ε/3. This conservative approach ensures that even if an attacker exploits correlations between sensitive words, the total privacy loss remains bounded.

### Formal Guarantees

The paper defines a security game for prompt sanitizers (similar to semantic security in cryptography) and proves that Prεεmpt satisfies this definition under the following conditions: the FPE scheme is pseudorandom, and the mDP mechanism satisfies ε-mLDP. For invariant prompts, the system satisfies an (α, α)-utility guarantee, meaning the quality score of the model response (in expectation) is equal before and after sanitization.

## Research Setup

**Hardware requirements**: The system is designed as a lightweight middleware. The computational overhead of FPE encryption and mDP noise addition is far lower than homomorphic encryption or secure multi-party computation. The paper notes that HE/MPC schemes require over 16 minutes for a single BERT inference, while Prεεmpt's operations are on the millisecond scale.

**Software dependencies**: A Named Entity Recognition (NER) module is needed to label sensitive words and their types. The paper uses predefined sanitizer rules for known formats (e.g., SSN, credit card numbers), and relies on NER models for types that require context (e.g., names).

**Evaluation setup**: Translation uses BLEU score; RAG tasks use accuracy; ConvFinQA uses relative error and prediction consistency. Model choices, dataset details, and the full evaluation protocol can be found in the original paper.

## Comprehensive Analysis

The smartest aspect of this paper is that it **abandons a one-size-fits-all privacy solution**. Previous work either used homomorphic encryption to protect everything (but too slow to be practical), or table-based replacement (but requires maintaining growing state), or direct LLM-based obfuscation (but without formal guarantees). Prεεmpt's choice is: classify first, then match the tool. Format-dependent types use FPE for perfect reversibility; value-dependent types use mDP for approximate protection. This "divide and conquer" approach allows the system to find an operable balance between privacy guarantees and practicality.

One **limitation** worth discussing: the accuracy of classification directly determines the system's effectiveness and security. If NER misclassifies "salary" as format-dependent, the FPE-encrypted salary number may have the correct format, but the model may not be able to give meaningful financial advice based on it—utility suffers. Conversely, if "SSN" is misclassified as value-dependent, mDP noise may be insufficient to hide the true value—privacy suffers. The paper assumes accurate classification, but in real deployment, this assumption requires additional verification mechanisms.

Another **boundary condition**: mDP's utility guarantee relies on the assumption that the model's response is insensitive to small perturbations of sensitive words. For tasks requiring precise numerical computation (e.g., "calculate exact after-tax income"), mDP noise will lead to incorrect answers. The (α, α)-utility guarantee only applies to the subset of "invariant prompts"—not all tasks satisfy this condition.

## Practical Applications

If you plan to integrate a Prεεmpt-like solution into your project, consider the following:

**Scenario selection**: Deploy first in tasks that are insensitive to exact sensitive values, such as document translation, financial document summarization, log analysis, and customer service dialogues. For tasks requiring precise numerical computation, carefully evaluate whether the mDP noise error is acceptable.

**Key management**: FPE reversibility means that if the key leaks, all historically sanitized data can be decrypted. Store keys in a Hardware Security Module (HSM) or OS keychain, not in plaintext configuration files.

**Classification accuracy monitoring**: After deploying NER classification, establish manual spot checks or automated verification to regularly check the accuracy of sensitive word classification. Misclassification is the most likely cause of system failure.

**Privacy budget tuning**: Smaller ε means stronger privacy but larger numerical perturbation. Start with the paper's recommended defaults and adjust gradually based on the utility sensitivity of your actual task. For scenarios involving financial or medical data, consider lowering ε appropriately.

**Integration with existing compliance frameworks**: Prεεmpt's stateless design naturally aligns with data minimization principles. Under GDPR, it can serve as part of the "technical measures" to demonstrate that the organization has taken reasonable protection before sending prompts to third-party LLMs.

