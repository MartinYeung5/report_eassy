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

## 🔬 Core Innovations

- **Boolean-first design**: Operates directly on Boolean circuits (bit-level AND/XOR) rather than encoding them into prime fields
- **Batch processing**: Optimized for large batches of standard cryptographic hash functions
- **Bitslicing**: Leverages SIMD parallelism by packing multiple Boolean inputs into registers
- **Trade-off strategy**: Sacrifices generality for extreme efficiency in targeted workloads

## 🎯 Applications

- Blockchain light clients
- Cross-chain bridges
- zk-Rollups
- Post-quantum signature schemes
- Verifiable computation for hash-dominated workloads

## ⚠️ Status

**Research prototype** — Not yet production-ready.

## 📚 References

- Authors: Benedikt Bünz (Espresso/NYU), Ron Rothblum (Succinct), William Wang (NYU)
- Paper: https://github.com/succinctlabs/flock/blob/main/paper/flock-paper.pdf
