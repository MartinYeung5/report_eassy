# Justitia-L: Budget-Constrained Fairness Optimization in Sharded Blockchains via Lagrangian Dual Control

## Paper Highlights

This paper proposes **Justitia-L**, a hierarchical adaptive incentive framework that addresses the persistent unfairness between cross-shard transactions (CTX) and intra-shard transactions (ITX) in sharded blockchains. Its core innovation lies in decoupling micro-level delay correction from macro-level budget management, introducing a **Justitia-PID closed-loop feedback control module** and **Lagrangian dual control** to ensure fairness for CTX while strictly constraining token inflation risk, thereby achieving economic sustainability.

---

## Core Research Contributions

### Problem Definition

In a sharded blockchain, a cross-shard transaction (CTX) must be split into two sub-transactions, which are queued separately in the transaction pools of the source shard and the target shard. However, the user pays only a single transaction fee for the original transaction, meaning the system must perform "two jobs" for the price of one. Driven by economic rationality, blockchain proposers naturally prefer to prioritize intra-shard transactions (ITX) that offer higher per‑unit returns, leaving CTX at a persistent competitive disadvantage with significantly longer queuing delays than ITX.

Although prior work (e.g., Justitia) has attempted to mitigate this issue via incentive mechanisms, its design relies primarily on **open‑loop or semi‑dynamic control**, which struggles to maintain control precision under fluctuating workloads. Moreover, it lacks strict economic guarantees against token inflation. Therefore, how to achieve fairness optimization under **budget constraints** becomes a problem of both theoretical depth and engineering challenge.

### Innovative Approach

Justitia‑L introduces innovations at three levels:

**First, a hierarchical decoupled architecture.** The problem is split into two layers: the micro‑layer is responsible for real‑time correction of delay disparity between CTX and ITX, while the macro‑layer handles global budget (token issuance) management and constraint enforcement. This separation allows the two objectives to be optimized independently, reducing problem complexity.

**Second, closed‑loop feedback control (Justitia‑PID).** At the micro‑layer, a PID (Proportional‑Integral‑Derivative) control law is employed to monitor the queuing delay gap between CTX and ITX in real time and dynamically adjust the incentive intensity. Unlike open‑loop designs, the closed‑loop controller can automatically correct control signals according to system state variations, maintaining high control accuracy under fluctuating loads.

**Third, Lagrangian dual control for budget management.** At the macro‑layer, the budget constraint is formulated as an optimization problem, and the Lagrangian dual method is used to derive the optimal token issuance strategy. This approach minimizes the inflationary impact on the economic system while ensuring fairness objectives, providing rigorous economic sustainability guarantees for the incentive mechanism.

### Research Outcomes

According to official information from HuangLab, Justitia‑L has been accepted at **IEEE ICDCS 2026** (a CCF‑B class international conference in distributed computing) with an acceptance rate of 18.59%. The keywords cover "blockchain sharding, Lagrangian dual control, tokenomics, economic sustainability." Although specific quantitative experimental data are not yet publicly available, from a theoretical contribution perspective, Justitia‑L successfully introduces **control theory** (PID control) and **optimization theory** (Lagrangian duality) into blockchain fairness—a domain traditionally dominated by game theory and mechanism design—achieving meaningful interdisciplinary innovation.

### Practical Deployment Potential

Justitia‑L shows promising prospects for real‑world deployment for three reasons:

- **Mature experimental infrastructure:** Its predecessor Justitia has already been prototyped and validated on the open‑source sharded blockchain emulator **BlockEmulator**, replaying over 500,000 real Ethereum transactions. BlockEmulator has been adopted by researchers from more than 80 countries and regions, providing a solid foundation for rapid validation and iteration of Justitia‑L.

- **Existing industry collaborations:** The underlying sharded blockchain infrastructure (BrokerChain) that underpins Justitia has already supported enterprise‑grade blockchain systems including Huawei Cloud Blockchain, Pengcheng Laboratory Blockchain, and Shanghai Insurance Exchange Blockchain. As an upgrade for fairness optimization, Justitia‑L is expected to follow this industry collaboration path.

- **Practicality of budget constraints:** Any public chain's economic model must face the hard constraint that "incentives cannot expand indefinitely." By incorporating budget constraints directly into the optimization framework, Justitia‑L becomes more feasible in real economic environments.

---

## Technical Details

### 1. Problem Formulation

Let there be $N$ shards in the system. At each time step $t$, CTX and ITX arrive at the transaction pools at rates $\lambda_{CTX}(t)$ and $\lambda_{ITX}(t)$, respectively. Define the fairness metric $F(t)$ as the ratio of average queuing delays between CTX and ITX:

$$F(t) = \frac{D_{CTX}(t)}{D_{ITX}(t)}$$

Ideally $F(t) \rightarrow 1$, meaning the queuing delays of the two transaction types converge.

### 2. Justitia‑PID Control Law

At the micro‑layer, the PID controller takes the fairness error $e(t) = F(t) - 1$ as input and outputs the incentive adjustment $u(t)$:

$$u(t) = K_p e(t) + K_i \int_0^t e(\tau)d\tau + K_d \frac{de(t)}{dt}$$

where $K_p$, $K_i$, and $K_d$ are the proportional, integral, and derivative coefficients, respectively. The controller output $u(t)$ determines the extra subsidy granted to CTX.

### 3. Lagrangian Dual Budget Optimization

The macro‑layer budget constraint can be formulated as the following optimization problem:

$$\min_{\theta} \mathcal{L}(\theta, \lambda) = \text{Penalty}(\theta) + \lambda \cdot (B(\theta) - B_{max})$$

where $\theta$ denotes the incentive strategy parameters, $B(\theta)$ is the expected token issuance, $B_{max}$ is the budget cap, and $\lambda \geq 0$ is the Lagrangian multiplier. By iteratively updating $\lambda$ via dual ascent, the fairness penalty can be minimized while satisfying the budget constraint.

### 4. System Stability Guarantee

The paper leverages the **Lyapunov optimization** framework to ensure long‑term system stability. By constructing a Lyapunov function $L(\Theta(t))$ and analysing its drift, it can be shown that under the Slater condition, the upper bound of the system queue length satisfies:

$$\lim_{t \to \infty} \sup \frac{1}{t} \sum_{\tau=0}^{t-1} \sum_{j=1}^N \mathbb{E}\{Q_j(\tau)\} \leq \frac{B + V(\text{pen}^* - \text{pen}^{min})}{\epsilon}$$

This result guarantees that under the budget constraint, shard queues do not grow indefinitely and the system remains stable.

---

## Experimental Setup

Based on the experimental configuration of the predecessor work Justitia, the study for Justitia‑L most likely adopts the following settings:

| Item | Specification |
|------|---------------|
| **Blockchain Protocol** | PBFT‑based sharded permissioned chain / state sharding (e.g., Monoxide) |
| **Experimental Platform** | BlockEmulator (open‑source sharded blockchain simulation platform) |
| **Dataset** | Historical Ethereum transaction data (≥ 500,000 transactions) |
| **Network Scale** | Variable number of shards $N$ (typically 4‑64) |
| **Evaluation Metrics** | CTX queuing delay, CTX confirmation ratio, token inflation rate, system throughput |
| **Baselines** | Monoxide protocol, original Justitia |

BlockEmulator is an open‑source blockchain emulator developed by HuangLab, built on the basic implementation framework of BrokerChain, and integrates built‑in algorithms and experimental toolkits from multiple research outcomes.

---

## Comprehensive Analysis

### Theoretical Contribution: Interdisciplinary Methodological Fusion

The most noteworthy theoretical contribution of Justitia‑L is its introduction of **control theory** (PID closed‑loop control) and **convex optimization theory** (Lagrangian duality) into the domain of blockchain fairness optimization. Previous work, including the original Justitia, primarily relied on game‑theoretic tools (e.g., Shapley value) to design incentive mechanisms. While these methods perform well in static analyses, they lack adaptive capabilities under dynamic and fluctuating workloads. Justitia‑L's closed‑loop feedback design addresses this gap, enabling the system to "sense" its own state and adjust in real time.

### From "Fairness" to "Sustainable Fairness"

The original Justitia aimed primarily to "treat CTX and ITX equally." Justitia‑L adds an additional dimension: **budget constraints**. This shift may appear small but is conceptually significant: it elevates the research perspective from "whether fairness can be achieved" to "whether fairness can be achieved **sustainably**." In any token economic model, unconstrained incentives inevitably lead to runaway inflation, undermining long‑term trust in the system. By incorporating economic sustainability as a first‑class constraint in the optimization framework, Justitia‑L moves closer to real‑world deployment requirements.

### Limitations and Challenges

Nevertheless, Justitia‑L faces several potential challenges. First, the optimal PID parameters ($K_p, K_i, K_d$) may vary under different network sizes and transaction loads; adaptive tuning remains an open problem. Second, while the Lagrangian dual method is theoretically elegant, collecting and reaching consensus on global budget information in a decentralized environment is itself a complex issue. Finally, the currently disclosed information remains largely theoretical, and detailed experimental data have yet to be made public.

---

## Practical Application Recommendations

For teams planning to deploy Justitia‑L in real sharded blockchain systems, the following recommendations may be useful:

**1. Start with simulation validation.** Replicate the experiments on BlockEmulator first to understand how PID parameters affect system behaviour, then gradually migrate to a production environment.

**2. Carefully set the budget parameter.** $B_{max}$ (the token issuance cap) is the critical link between fairness and economic sustainability. Consider the system's annual inflation target, transaction volume forecasts, node reward expectations, and other factors when determining this value.

**3. Deploy incrementally.** Enable the Justitia‑L incentive mechanism on a subset of shards or transaction types initially, observe its actual impact on latency, throughput, and token inflation, and then expand the scope gradually.

**4. Leverage existing sharding infrastructure.** Justitia‑L inherits from the BrokerChain technology stack, which already supports enterprise systems such as Huawei Cloud Blockchain and Pengcheng Laboratory Blockchain. If your system is built on these foundations, integration costs will be significantly lower.

---

## References

- Sihua Wang, Jian Zheng, Huawei Huang. “Justitia-L: Budget-Constrained Fairness Optimization in Sharded Blockchains via Lagrangian Dual Control.” *IEEE International Conference on Distributed Computing Systems (ICDCS)*, 2026.

- Jian Zheng, Huawei Huang, Yinqiu Liu, Taotao Li, Hong-Ning Dai, Zibin Zheng. “Justitia: An Incentive Mechanism towards the Fairness of Cross-shard Transactions.” *IEEE INFOCOM* 2025.

- HuangLab official introduction: *Three blockchain papers from HuangLab accepted at ICDCS 2026*.
