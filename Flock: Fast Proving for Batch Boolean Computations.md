# 🦅 Flock: Fast Proving for Batch Boolean Computations

> **Breaking the Speed Limit on SNARK Proving** — Flock is a SNARK system designed specifically for batch Boolean computations, achieving unprecedented performance for standard hash functions (Keccak, SHA-256, BLAKE3).

## 📊 Key Performance Metrics

| Metric | BLAKE3 | SHA-256 | Keccak-f[1600] |
|--------|--------|---------|----------------|
| **Throughput (single-core)** | 82,100/s | 42,100/s | 30,700/s |
| **Throughput (10-core)** | 660,000/s | — | — |
| **Proof Size** | <450 KB | <450 KB | <450 KB |
| **Verification Time** | <4 ms | <4 ms | <4 ms |
| **Proof/Compute Overhead** | ~250× | ~170× | ~245× |

## 🏆 Performance vs. State-of-the-Art

| Comparison | Speedup |
|------------|---------|
| SHA-256 vs Binius64 | **8.4×** |
| BLAKE3 vs Binius64 | **14×** |
| BLAKE3 vs Plonky3 | **14×** |
| Keccak vs Hashcaster (multi-thread) | **4×** |
| vs Plonky3 (prime-field systems) | **10–40×** |
| vs Spartan | **100×+** |

## 🔬 Core Innovations & Research Highlights

### 1. Core Problem & Motivation
- **Background**: SNARK proving is 10,000 to 1,000,000× more expensive than native computation in general zkVMs.
- **Target**: Drastically reduce the proving overhead for batch standard hash functions (Keccak, SHA-256, BLAKE3) which form the bedrock of blockchain and cryptographic applications.
- **Limitations of Prior Work**:
  - **Arithmetic Field Mismatch**: Standard hashes are Boolean circuits (AND/XOR), but encoding them into prime fields introduces massive overhead.
  - **Generality Penalty**: Universal zkVMs are four to six orders of magnitude slower than native execution for hash-heavy workloads.
  - **Poor Hash Proving**: Existing prime-field systems (e.g., Plonky3) are 10–40× slower than Flock; Spartan is hundreds of times slower.

### 2. Key Results & Findings
- Flock achieves a **Proof/Compute overhead ratio of just ~170× to ~250×**—compressing the gap from six orders of magnitude down to two.
- **Bitcoin Block Proving**: Completes post-quantum signature hash proofs in **2.5 seconds**.
- **Scalability**: Easily parallelizable; 10-core throughput exceeds 660,000 BLAKE3 compressions per second.

### 3. Deep Insights & Implications
- **Boolean-first Paradigm**: Flock operates directly on Boolean circuits instead of encoding them into prime fields, proving that "going back to Boolean" is the correct path for hash-intensive workloads.
- **Bitslicing Power**: Packs multiple Boolean inputs into large registers, using single bitwise instructions to perform multiple parallel Boolean operations—translating hardware parallelism into proving acceleration.
- **Paradigm Shift**: SNARK design is shifting from "arithmetic-first" to "computation-pattern-first." Flock validates that sacrificing generality for extreme efficiency is a worthwhile trade-off for targeted workloads.

### 4. Conclusions & Future Work
- **Main Conclusion**: By designing SNARKs directly on Boolean circuits and focusing on batch standard hashing, unprecedented proving efficiency on consumer hardware is achievable.
- **Current Status**: **Research prototype** — not yet production-ready.
- **Future Directions**:
  - Apply Flock's core ideas to general-purpose zkVMs (hybrid architectures).
  - Evolve from prototype to production-grade systems.
  - Extend to more types of batch Boolean circuit proving (e.g., matrix multiplication, FFT, signature verification).

---

## 🎯 Recommended Applications

- Blockchain Light Clients
- Cross-chain Bridges
- zk-Rollups
- Post-Quantum Signature Schemes (e.g., hash-based signatures)
- Verifiable Computation for Hash-Dominated Workloads

---

## ⚠️ Limitations & Boundaries

- **Specialization Constraint**: Fully abandons generality; only applicable to batch fixed Boolean circuits.
- **Hardware Dependency**: Performance benchmarks are based on Apple M4 Max; results may vary across platforms.
- **Production Readiness**: Currently a research prototype, not yet hardened for production deployment.

---

## 📚 References

- **Authors**: Benedikt Bünz (Espresso/NYU), Ron Rothblum (Succinct), William Wang (NYU)
- **Paper Title**: *Flock: Fast Proving for Batch Boolean Computations*
- **Code Repository**: [github.com/succinctlabs/flock](https://github.com/succinctlabs/flock)

---

---

# 🦅 Flock：批量布爾計算的快速證明——深度分析總結

> **突破 SNARK 證明的速度極限** — Flock 是一個專為批量布爾計算設計的 SNARK 系統，在標準哈希函數（Keccak、SHA-256、BLAKE3）上實現了前所未有的效能表現。本倉庫收錄了該論文的深度分析摘要。

---

## 📊 核心性能指標（Apple M4 Max）

| 指標 | BLAKE3 | SHA-256 | Keccak-f[1600] |
|--------|--------|---------|----------------|
| **單核吞吐量（每秒）** | 82,100 | 42,100 | 30,700 |
| **十核吞吐量（每秒）** | 660,000+ | — | — |
| **證明大小** | <450 KB | <450 KB | <450 KB |
| **驗證時間** | <4 毫秒 | <4 毫秒 | <4 毫秒 |
| **證明/計算開銷比** | ~250× | ~170× | ~245× |

---

## 🏆 與現有技術的效能對比

| 比較項 | 加速比 |
|---------|------------|
| SHA-256 vs Binius64 | **快約 8.4 倍** |
| BLAKE3 vs Binius64 | **快約 14 倍** |
| BLAKE3 vs Plonky3 | **快約 14 倍** |
| Keccak vs Hashcaster（多執行緒） | **快約 4 倍** |
| vs 基於素數域系統（Plonky3） | **快約 10–40 倍** |
| vs Spartan | **快數百倍** |

---

## 🔬 核心創新與研究重點

### 1. 核心問題與動機
- **研究背景**：在通用 zkVM 中，證明的成本可比原生計算高出 **一萬到一百萬倍**。
- **目標**：大幅降低批量標準哈希函數（Keccak、SHA-256、BLAKE3）的證明開銷，這些函數是區塊鏈與密碼學應用的基石。
- **既有研究的局限性**：
  - **算術域失配**：標準哈希本質上是布爾電路（AND/XOR），但將其編碼到素數域中會引入巨大的額外開銷。
  - **通用性代價**：通用 zkVM 在哈希密集型工作負載下，比原生執行慢四到六個數量級。
  - **哈希證明效率低落**：既有基於素數域的系統（如 Plonky3）比 Flock 慢 10–40 倍；Spartan 則慢了數百倍。

### 2. 關鍵結果與發現
- Flock 將 **證明/計算開銷比** 壓縮至約 **170 倍至 250 倍**——將差距從六個數量級縮小到兩個數量級。
- **比特幣區塊證明**：完成後量子簽名哈希證明的時間僅需 **2.5 秒**。
- **可擴展性**：輕鬆實現並行化，十核心吞吐量超過每秒 660,000 次 BLAKE3 壓縮。

### 3. 深度洞見與影響
- **布爾電路優先的設計哲學**：Flock 直接基於布爾電路運作，而非將其編碼到素數域中，證明了「回歸布爾」是哈希密集型工作負載的正確路徑。
- **Bitslicing 技術的威力**：將多個布爾輸入打包到大型暫存器中，用單一位元指令完成多個並行布爾操作——將硬體並行性直接轉化為證明加速。
- **範式轉移**：SNARK 設計正從「算術優先」轉向「計算模式優先」。Flock 驗證了為特定工作負載放棄通用性以換取極致效率，是非常值得的權衡。

### 4. 結論與後續工作
- **主要結論**：透過直接基於布爾電路設計 SNARK，並專注於批量標準哈希證明，可以在消費級硬體上實現前所未有的證明效率。
- **當前狀態**：**研究原型** — 尚未準備好用於生產環境。
- **後續規劃**：
  - 將 Flock 的核心思想應用於通用 zkVM（混合架構）。
  - 從研究原型演進為生產級系統。
  - 擴展到更多類型的批量布爾電路證明（如矩陣乘法、FFT、簽名驗證）。

---

## 🎯 建議應用場景

- 區塊鏈輕客戶端
- 跨鏈橋
- zk-Rollup
- 後量子簽章方案（如基於哈希的簽章）
- 哈希主導工作負載的可驗證計算

---

## ⚠️ 限制與邊界條件

- **專用性限制**：完全放棄通用性，僅適用於批量固定布爾電路。
- **硬體依賴**：效能數據基於 Apple M4 Max，不同硬體平臺上的表現可能有顯著差異。
- **生產就緒性**：目前仍為研究原型，尚未經過生產環境的嚴格考驗。

---

## 📚 參考文獻

- **作者**：Benedikt Bünz (Espresso/NYU)、Ron Rothblum (Succinct)、William Wang (NYU)
- **論文名稱**：《Flock: Fast Proving for Batch Boolean Computations》
- **程式碼倉庫**：[github.com/succinctlabs/flock](https://github.com/succinctlabs/flock)
