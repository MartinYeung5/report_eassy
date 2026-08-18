# Indicator of Benignity: An Industry View of False Positive in Malicious Domain Detection and Its Mitigation

## Key Points

This paper presents the first large‑scale empirical study of false positives (FPs) in malicious domain detection systems, based on 123,491 user‑reported FP cases collected from a global security vendor (anonymised as "SV") between 2019 and 2024. The study reveals fundamental limitations of popularity‑based whitelisting: even merging all public top‑domain lists covers at most 38% of FP domains. Expanding whitelists further would inevitably introduce an unacceptable number of malicious domains, making the trade‑off counterproductive.

---

## Core Research Content

### Problem Definition

Malicious domain detection is a cornerstone of cyber defence, yet the problem of false positives has long been neglected in academic research. Most studies focus on improving true positive rates, while the real‑world scale, distribution, and mitigation of FPs remain poorly understood. Given that the fear of false positives is one of the main barriers to deploying security systems in production, this research gap urgently needs to be addressed.

### Innovative Methods

The core innovation is a **first‑of‑its‑kind measurement study based on real‑world FP reports** from a production environment. The data come from firewalls deployed across more than 65,000 organisations worldwide, processing ~7 billion DNS queries daily and detecting ~1.6 million new malicious domains each day. When users (primarily SOC analysts) suspect a false positive, they submit a report; SV security researchers manually verify each case, providing reliable ground‑truth labels. The team analysed 123,491 reports collected over six years, examining temporal patterns, domain characteristics, and detection sources.

In addition, the paper introduces an **attribution scheme** that distinguishes FPs originating from SV’s in‑house detectors (PD) from those sourced from third‑party threat intelligence feeds (PF), enabling finer‑grained comparative analysis.

### Research Findings

Key findings include:

1. **Long‑tail distribution.** 97.7% of FP domains were reported by only a single user, and the vast majority reside under unique root domains. This means bulk remediation via root‑domain grouping is ineffective.

2. **Limited coverage of popularity whitelists.** Even merging all public top‑domain lists (Alexa, Majestic Million, Cisco Umbrella, Quantcast, CrUX) covers only ~38% of FP root domains. Notably, ~49.2% of reported FP subdomains have a root that appears in these lists, but the subdomains themselves were abused by attackers. Further analysis shows that ~50% of FP root domains rank below 5 million in the Chrome User Experience Report (CrUX). Extending whitelists to 5M or 10M entries would cover more FPs but inevitably introduce massive false negatives.

3. **Significant reporting latency.** About 10% of FPs are reported within one week of detection, ~23% within 30 days, but half take more than 120 days to be reported. This implies that evaluating a detector’s false‑positive rate requires at least 4 months of production deployment. Moreover, earlier reports are more likely to be confirmed as genuine FPs.

4. **Malicious‑website detectors are the primary FP source.** Among SV’s in‑house detectors, malicious‑website detection contributed 87,596 FP reports (the vast majority), while DGA detectors (~11 million daily detections, only 1,130 FPs) and domain‑shadowing detectors (zero FPs) proved much more accurate. Further investigation revealed that most reported FP domains belong to small business websites that were indeed compromised at the time of detection but had been cleaned up by the time the user complained.

### Practical Deployment Potential

The findings offer immediate engineering guidance. The long‑tail distribution and the TODTOC (Time of Detection to Time of Complaint) analysis framework can serve as quantitative tools for security vendors to assess detector quality and optimise FP handling workflows. The quantitative assessment of whitelist coverage provides empirical evidence against the common but flawed practice of blindly expanding whitelists. Finally, the observation that FPs often stem from compromised‑then‑cleaned small business sites suggests that vendors could explore dynamic risk assessment based on historical security states, rather than relying solely on static popularity rankings.

---

## Technical Details

### Attribution Scheme

FP reports are attributed to two detection sources:

- **PD (Produced by SV’s own detectors):** The domain was flagged by SV’s in‑house systems.
- **PF (Produced by third‑party feeds):** The domain appeared only in external threat intelligence sources.

Further, FPs are classified by detection scope, including general malicious domain detection, domain squatting, DGA, DNS rebinding, fast‑flux, DNS tunnelling, and malicious‑website detection.

### Whitelist Coverage Calculation

Five public top‑domain lists are used:

1. Alexa top‑1m (discontinued in May 2022)
2. Majestic Million
3. Cisco Umbrella 1 million
4. Quantcast top 490K
5. Chrome User Experience Report (CrUX)

For each list, the root domain is extracted from the FQDN and matched. Coverage is defined as the proportion of FP root domains that appear in at least one list during the observation period.

### TODTOC Metric

TODTOC (Time of Detection to Time of Complaint) is the number of days between a domain being flagged as malicious and the user submitting a false‑positive complaint. This metric quantifies the delay in FP discovery and informs deployment evaluation periods.

---

## Study Configuration

### Data Sources

- **Time span:** 2019 – 2024 (6 years)
- **Data volume:** 123,491 FP reports, of which 121,073 were accepted (corresponding to 118,093 unique FQDNs) and 2,418 rejected (2,022 unique FQDNs)
- **Detection scale:** Dozens of detectors; ~7 billion DNS queries processed daily; ~1.6 million new malicious domains detected each day
- **Coverage:** Firewalls at more than 65,000 organisations worldwide

### Ground‑Truth Acquisition

Reports are submitted by users (mainly SOC analysts) and manually verified by SV security researchers. Each report includes a “justification of benignity”, and combined with the researcher’s validation, the dataset achieves a high acceptance rate (>98%), indicating high trustworthiness of user‑reported FPs.

### Hardware/Software Environment

The paper does not disclose specific hardware details, but given the nature of the study:

- Storage and processing must support PB‑scale DNS logs with long‑term retention and query capabilities.
- The detection system is distributed, handling 7 billion queries per day in real time.
- Analysis tools likely include large‑scale data processing frameworks (e.g., Apache Spark) and statistical packages.

---

## Comprehensive Analysis

This paper makes significant contributions to both academic research and industrial practice.

**From a research‑paradigm perspective,** it exemplifies a critical but often overlooked direction – **systematic empirical studies in production environments**. Many academic papers propose novel detection algorithms and evaluate them on laboratory datasets, but their real‑world FP performance often diverges dramatically. The authors bridge this “academia‑industry gap” through close collaboration with a major vendor, providing invaluable first‑hand data that will benefit future research.

**On the nature of the FP problem,** the paper reveals a deep tension: FP mitigation has relied too heavily on a single signal – popularity – but FP domains are inherently long‑tailed. This means that popularity‑based whitelisting is mathematically incapable of covering most FPs without introducing unacceptable false negatives. The CrUX data clearly show that ~50% of FP roots rank below 5 million, so expanding the whitelist five‑fold or more would inevitably let in many malicious domains. This finding delivers a strong challenge to the industry’s habitual “stack‑more‑whitelists” approach.

**From a temporal perspective,** the TODTOC distribution highlights a subtle but crucial issue: the social cost of FPs is decoupled from the “immediacy” of detection systems. While detectors pursue zero‑day coverage, user perception and feedback are often delayed. This means that a detector appearing to have zero FPs in the short term may generate a flood of complaints months later. The authors’ recommendation of a minimum 4‑month evaluation window is highly actionable.

**On the sources of FPs,** the dominance of malicious‑website detectors – and the fact that many FPs stem from compromised‑then‑cleaned sites – shows that FPs are not always detector “errors”. They often reflect **temporal mismatches** between detection and complaint. This suggests the need for finer‑grained domain state tracking and reputation decay mechanisms.

**Methodological limitations** should also be acknowledged. The data come from a single vendor; although large, the generalisability of findings across different vendors’ detectors, user bases, and labelling processes remains to be validated. The rejection rate of <2% indicates that almost all user complaints are accepted – this may reflect a lenient verification policy, or that users only report when very confident, but in any case it underscores the real and serious impact of FPs on users.

---

## Practical Applications

### For Security Vendors

1. **Re‑evaluate whitelisting strategies.** Since popularity covers at most 38% of FPs, vendors should consider multi‑dimensional “indicators of benignity” – e.g., historical security state, site owner reputation, certificate transparency logs, search engine indexing – to build more robust FP filtering mechanisms.

2. **Establish fast‑track FP feedback channels.** Early‑reported FPs are more likely to be accepted. Vendors should reduce the friction of submitting FP reports and create rapid verification and remediation workflows to shorten the impact window.

3. **Optimise malicious‑website detectors specifically.** As the largest FP source, and given the “compromised‑then‑cleaned” scenario, vendors could introduce time‑sensitive factors (e.g., auto‑clear alerts for cleaned sites) or apply differential thresholds for small‑business websites.

4. **Extend production evaluation periods for new detectors.** With at least 4 months needed to collect half of FPs, vendors should design longer observation phases before finalising FP rate assessments.

### For Academic Research

1. **Value production data.** This paper demonstrates the unique value of industrial data. Researchers should seek collaborations to obtain real‑world labelled datasets, rather than relying solely on synthetic or laboratory data.

2. **Explore beyond‑popularity FP mitigation.** Given the structural limitations of popularity whitelists, research should investigate FP identification based on behavioural features, registration patterns, content semantics, and other dimensions.

3. **Study the temporal dynamics of FPs.** The “temporal mismatch” phenomenon raises interesting questions: how can we model the evolution of domain security states and design confidence‑decay mechanisms?

### For End‑Users (Enterprises)

1. **Report FPs promptly.** Early reporting increases the chance of confirmation and faster remediation. SOC teams should submit reports as soon as they suspect an FP.

2. **Strengthen website security.** Many FPs originate from sites that were compromised and later cleaned. Enterprises should invest in proactive defences – regular vulnerability scanning, WAF deployment, timely patching – to reduce the likelihood of being flagged in the first place.

---

## References

- Original paper: [https://www.ndss-symposium.org/wp-content/uploads/2026-f1869-paper.pdf](https://www.ndss-symposium.org/wp-content/uploads/2026-f1869-paper.pdf)
