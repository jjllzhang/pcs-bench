# FRI-Binius / Plonky3 `p3-binary-pcs` Benchmark Results

Measurements of [Plonky3 `p3-binary-pcs`](https://github.com/Plonky3/Plonky3) (`BinaryPcs` over additive Cantor-domain Reed–Solomon with BaseFold-style folding, and `BooleanPcs` with bit-packing + FRI-Binius-style ring switching) for `k` from 20 to 30.

> **Naming & Protocol Note**: This codebase implements Binius/FRI-Binius techniques inside Plonky3 (`p3-binary-pcs`), rather than a standalone paper-exact FRI-Binius reference artifact.

## Setup & Parameters

- **Hardware & Runtime**: AMD EPYC 9754, 503 GiB RAM, 32 physical cores pinned to CPUs 0–31 (`RAYON_NUM_THREADS=32`); optimized release build (`target-cpu=native`, `parallel` feature enabled)
- **Field & Challenges**:
  - `BinaryPcs` (`gf32`, `gf64`, `gf128`): `BinaryField32`, `BinaryField64`, or `BinaryField128` base elements with `BinaryField128` challenges; `k` denotes `2^k` **base-field elements**.
  - `BooleanPcs` (`boolean`): `GF(2)` bits packed 128 per `BinaryField128` element (with ring switching and inner `BinaryPcs<BinaryField128, BinaryField128>`); `k` denotes `2^k` **bits** (i.e., `2^(k-7)` packed `GF(2^128)` elements).
- **Hash / Transcript**: `Keccak256Hash` Merkle tree (`MerkleTreeMmcs`) and `BinaryChallenger` (`HashChallenger<Keccak256Hash>`)
- **Security Target**: Target 100 bits (unique-decoding algebraic security bound via `BinaryPcsRegime` / `BooleanPcs::opening_security`, excluding hash/transcript collision resistance). Where `BinaryPcsConfig` rejects 100 bits, the highest accepted integer target `<= 100` is used and reported in `algebraic_bits` (dropping to `94.518` bits at `k = 30, rate = 1/2`). `BooleanPcs` reaches `100.44`–`102.39` algebraic bits (including ring switching).
- **Scheme Constants**: Code rates `1/2` and `1/4`; `pow_bits = 8` and `pow_bits = 0` for `BinaryPcs`, and `pow_bits = 0` for `BooleanPcs`; single-point opening at Boolean hypercube index `⌊2^k / 3⌋`.

## Measurement Methodology

- **Sampling**: 1 warmup followed by 3 measured runs per configuration; reported timings are **arithmetic means** across the 3 measured runs.
- **Timing Scope**: `Commit` includes transcript observation; `Open` excludes commitment; `Prover` is `Commit + Open` (mean) computed at full precision prior to rounding; `Verify` excludes replaying the commitment. `Peak RSS` is per-configuration process peak resident memory (including warmup and verification checks).
- **Proof Size**: Reported in KiB (`1 KiB = 1024 bytes`) rounded to 3 decimal places from `postcard` serialization of the `proof` object; **excludes** the separately transmitted commitment (and for `BooleanPcs`, also excludes the separately transmitted evaluation value).
- **Verification & Missing Values**: Every executed configuration passed verification and rejected corrupted claims. `—` denotes unmeasured runs skipped prior to execution due to predicted memory overflow (`SKIPPED_MEMORY` at `k = 30, rate = 1/4` for `BinaryPcs`, estimated at `685.4 GiB`, `708.3 GiB`, and `754.4 GiB` vs. `~428 GiB` safe machine limit).

## Results

### BinaryPcs — GF(2^32) base, GF(2^128) challenges (Rate 1/2, PoW 8)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.034 | 0.076 | 0.110 | 6.531 | 672.767 | 0.271 | 101.326 |
| 22 | 0.090 | 0.203 | 0.293 | 4.631 | 863.276 | 1.150 | 101.166 |
| 24 | 0.268 | 0.724 | 0.992 | 5.494 | 1098.293 | 4.676 | 100.659 |
| 26 | 1.031 | 2.782 | 3.813 | 6.302 | 1319.807 | 18.772 | 98.613 |
| 28 | 4.203 | 11.040 | 15.243 | 7.360 | 1578.516 | 75.123 | 96.566 |
| 30 | 17.488 | 44.465 | 61.953 | 8.107 | 1829.793 | 300.465 | 94.518 |

### BinaryPcs — GF(2^32) base, GF(2^128) challenges (Rate 1/4, PoW 8)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.049 | 0.083 | 0.133 | 2.821 | 497.352 | 0.560 | 101.446 |
| 22 | 0.139 | 0.335 | 0.474 | 3.423 | 644.479 | 2.307 | 101.117 |
| 24 | 0.520 | 1.260 | 1.780 | 3.863 | 771.117 | 9.295 | 99.557 |
| 26 | 2.100 | 4.999 | 7.099 | 4.696 | 927.559 | 37.243 | 97.535 |
| 28 | 8.340 | 19.864 | 28.204 | 5.473 | 1101.197 | 148.996 | 95.513 |
| 30 | — | — | — | — | — | — | — |

### BinaryPcs — GF(2^64) base, GF(2^128) challenges (Rate 1/2, PoW 8)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.034 | 0.073 | 0.108 | 3.845 | 672.987 | 0.285 | 101.326 |
| 22 | 0.091 | 0.178 | 0.269 | 4.639 | 870.802 | 1.196 | 101.166 |
| 24 | 0.295 | 0.694 | 0.989 | 5.510 | 1097.838 | 4.864 | 100.659 |
| 26 | 1.219 | 2.644 | 3.863 | 6.583 | 1318.451 | 19.511 | 98.613 |
| 28 | 4.573 | 10.481 | 15.054 | 7.174 | 1570.637 | 78.096 | 96.566 |
| 30 | 20.297 | 42.067 | 62.364 | 8.015 | 1814.600 | 312.483 | 94.518 |

### BinaryPcs — GF(2^64) base, GF(2^128) challenges (Rate 1/4, PoW 8)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.056 | 0.104 | 0.160 | 2.873 | 506.356 | 0.582 | 101.446 |
| 22 | 0.148 | 0.336 | 0.484 | 3.414 | 644.407 | 2.387 | 101.117 |
| 24 | 0.568 | 1.228 | 1.796 | 3.964 | 794.318 | 9.609 | 99.557 |
| 26 | 2.200 | 4.841 | 7.042 | 4.534 | 939.159 | 38.484 | 97.535 |
| 28 | 9.215 | 19.458 | 28.672 | 5.962 | 1079.356 | 153.977 | 95.513 |
| 30 | — | — | — | — | — | — | — |

### BinaryPcs — GF(2^128) base, GF(2^128) challenges (Rate 1/2, PoW 8)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.031 | 0.097 | 0.128 | 3.796 | 669.703 | 0.317 | 101.326 |
| 22 | 0.080 | 0.199 | 0.279 | 4.622 | 887.666 | 1.328 | 101.166 |
| 24 | 0.258 | 0.743 | 1.001 | 5.447 | 1097.096 | 5.355 | 100.659 |
| 26 | 1.148 | 2.919 | 4.068 | 6.411 | 1314.683 | 21.440 | 98.613 |
| 28 | 4.176 | 11.578 | 15.753 | 7.295 | 1567.655 | 85.778 | 96.566 |
| 30 | 17.273 | 46.481 | 63.754 | 8.317 | 1855.476 | 343.119 | 94.518 |

### BinaryPcs — GF(2^128) base, GF(2^128) challenges (Rate 1/4, PoW 8)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.042 | 0.103 | 0.145 | 2.840 | 503.599 | 0.623 | 101.446 |
| 22 | 0.140 | 0.379 | 0.519 | 3.337 | 635.686 | 2.552 | 101.117 |
| 24 | 0.501 | 1.324 | 1.825 | 3.809 | 780.068 | 10.248 | 99.557 |
| 26 | 2.379 | 5.249 | 7.628 | 4.603 | 944.567 | 40.994 | 97.535 |
| 28 | 8.503 | 21.018 | 29.521 | 5.508 | 1115.980 | 163.993 | 95.513 |
| 30 | — | — | — | — | — | — | — |

### BinaryPcs — GF(2^32) base, GF(2^128) challenges (Rate 1/2, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.032 | 0.057 | 0.089 | 4.061 | 719.097 | 0.275 | 101.216 |
| 22 | 0.084 | 0.200 | 0.284 | 4.889 | 922.263 | 1.148 | 101.067 |
| 24 | 0.281 | 0.814 | 1.095 | 5.881 | 1179.191 | 4.674 | 100.589 |
| 26 | 1.035 | 2.952 | 3.986 | 6.865 | 1419.037 | 18.773 | 98.541 |
| 28 | 4.011 | 11.234 | 15.244 | 7.901 | 1677.760 | 75.119 | 96.493 |
| 30 | 17.300 | 44.746 | 62.046 | 8.647 | 1961.042 | 300.481 | 94.444 |

### BinaryPcs — GF(2^32) base, GF(2^128) challenges (Rate 1/4, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.047 | 0.082 | 0.129 | 3.006 | 538.208 | 0.560 | 100.943 |
| 22 | 0.135 | 0.367 | 0.502 | 3.587 | 682.884 | 2.307 | 100.704 |
| 24 | 0.547 | 1.391 | 1.937 | 4.167 | 839.939 | 9.294 | 99.642 |
| 26 | 1.987 | 5.188 | 7.174 | 4.936 | 1009.907 | 37.247 | 97.621 |
| 28 | 7.986 | 20.052 | 28.038 | 5.886 | 1187.872 | 148.995 | 95.599 |
| 30 | — | — | — | — | — | — | — |

### BinaryPcs — GF(2^64) base, GF(2^128) challenges (Rate 1/2, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.033 | 0.066 | 0.099 | 4.040 | 718.299 | 0.285 | 101.216 |
| 22 | 0.092 | 0.204 | 0.297 | 4.971 | 939.478 | 1.203 | 101.067 |
| 24 | 0.303 | 0.775 | 1.078 | 5.886 | 1178.835 | 4.862 | 100.589 |
| 26 | 1.200 | 2.793 | 3.993 | 6.707 | 1417.125 | 19.517 | 98.541 |
| 28 | 4.532 | 10.682 | 15.215 | 7.902 | 1702.531 | 78.094 | 96.493 |
| 30 | 19.746 | 42.519 | 62.265 | 8.643 | 1954.491 | 312.461 | 94.444 |

### BinaryPcs — GF(2^64) base, GF(2^128) challenges (Rate 1/4, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.056 | 0.100 | 0.156 | 3.031 | 538.382 | 0.580 | 100.943 |
| 22 | 0.151 | 0.382 | 0.533 | 3.627 | 689.696 | 2.386 | 100.704 |
| 24 | 0.576 | 1.390 | 1.966 | 4.225 | 850.771 | 9.610 | 99.642 |
| 26 | 2.230 | 5.067 | 7.297 | 4.912 | 1007.222 | 38.486 | 97.621 |
| 28 | 9.387 | 19.711 | 29.098 | 5.557 | 1194.426 | 153.980 | 95.599 |
| 30 | — | — | — | — | — | — | — |

### BinaryPcs — GF(2^128) base, GF(2^128) challenges (Rate 1/2, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.030 | 0.085 | 0.115 | 4.028 | 716.233 | 0.316 | 101.216 |
| 22 | 0.085 | 0.221 | 0.306 | 4.934 | 938.390 | 1.323 | 101.067 |
| 24 | 0.270 | 0.833 | 1.102 | 5.837 | 1177.153 | 5.355 | 100.589 |
| 26 | 1.220 | 3.119 | 4.339 | 6.919 | 1419.749 | 21.436 | 98.541 |
| 28 | 4.197 | 11.801 | 15.998 | 7.615 | 1685.253 | 85.772 | 96.493 |
| 30 | 16.945 | 46.507 | 63.452 | 8.670 | 1984.378 | 343.116 | 94.444 |

### BinaryPcs — GF(2^128) base, GF(2^128) challenges (Rate 1/4, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.046 | 0.113 | 0.159 | 3.020 | 545.809 | 0.626 | 100.943 |
| 22 | 0.146 | 0.402 | 0.548 | 3.517 | 678.341 | 2.550 | 100.704 |
| 24 | 0.541 | 1.477 | 2.018 | 4.266 | 844.442 | 10.246 | 99.642 |
| 26 | 2.279 | 5.410 | 7.690 | 4.894 | 1021.392 | 40.996 | 97.621 |
| 28 | 8.583 | 21.187 | 29.770 | 5.489 | 1197.831 | 163.998 | 95.599 |
| 30 | — | — | — | — | — | — | — |

### BooleanPcs — GF(2) bits packed into GF(2^128) (Rate 1/2, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.002 | 0.004 | 0.006 | 1.895 | 220.215 | 0.006 | 102.098 |
| 22 | 0.005 | 0.006 | 0.011 | 2.435 | 330.060 | 0.004 | 102.096 |
| 24 | 0.010 | 0.016 | 0.025 | 3.087 | 461.446 | 0.018 | 102.087 |
| 26 | 0.020 | 0.052 | 0.072 | 4.025 | 640.053 | 0.149 | 102.052 |
| 28 | 0.058 | 0.099 | 0.158 | 4.644 | 842.932 | 0.654 | 101.918 |
| 30 | 0.152 | 0.337 | 0.489 | 5.545 | 1069.032 | 2.678 | 101.480 |

### BooleanPcs — GF(2) bits packed into GF(2^128) (Rate 1/4, PoW 0)

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) | Algebraic Bits |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.004 | 0.005 | 0.009 | 1.553 | 184.332 | 0.002 | 102.387 |
| 22 | 0.006 | 0.008 | 0.015 | 1.896 | 258.246 | 0.004 | 102.381 |
| 24 | 0.014 | 0.017 | 0.031 | 4.335 | 356.563 | 0.070 | 102.360 |
| 26 | 0.030 | 0.051 | 0.081 | 2.877 | 487.749 | 0.308 | 102.275 |
| 28 | 0.091 | 0.173 | 0.263 | 3.419 | 611.020 | 1.269 | 101.980 |
| 30 | 0.287 | 0.620 | 0.907 | 4.072 | 765.188 | 5.110 | 100.437 |

## Scheme-Specific Parameters & Notes

- **Input size comparability**: Under the same `k`, `gf32`, `gf64`, and `gf128` represent `2^k` field elements (`4·2^k`, `8·2^k`, and `16·2^k` bytes respectively), whereas `boolean` represents `2^k` bits (`2^(k-3)` bytes, packed into `2^(k-7)` `GF(2^128)` elements). Do not compare `BinaryPcs` and `BooleanPcs` directly at equal `k` without accounting for input byte size.
- **Skipped runs (`SKIPPED_MEMORY`)**: `gf32-rate-1/4`, `gf64-rate-1/4`, and `gf128-rate-1/4` at `k = 30` were skipped before running based on `k = 28` peak memory extrapolation (`685.4 GiB`, `708.3 GiB`, `754.4 GiB` > `~428 GiB` safe limit), rather than failing with a runtime OOM crash.
- **Queries & configured targets**: Per-configuration `queries`, `pow_bits`, `configured_bits`, `algebraic_bits`, and `status` are recorded in [`fri-binius.csv`](fri-binius.csv).
