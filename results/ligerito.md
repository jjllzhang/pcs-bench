# Ligerito Benchmark Results

Measurements of a locally modified version of [ligerito](https://github.com/rotkonetworks/ligerito) for `k` from 20 to 30 at code rates `1/2` and `1/4`.

## Setup & Parameters

- **Hardware & Runtime**: AMD EPYC 9754, 503 GiB RAM, 32 physical cores pinned to CPUs 0-31 (32 Rayon threads); optimized release build (`target-cpu=native`, LTO, single codegen unit)
- **Field & Challenges**: `BinaryElem32` ($\mathbb{F}_{2^{32}}$) input evaluations, with `BinaryElem128` ($\mathbb{F}_{2^{128}}$) opening-point coordinates, claimed values, and extension challenges
- **Hash / Transcript**: BLAKE3 Merkle hashing (`ligerito-merkle`); SHA-256 Fiat-Shamir transcript (`FiatShamir::new_sha256`)
- **Security Target**: At least 100 bits under the project's draft opening-inclusive estimate (`analyze_evaluation_openings(&vc, 1).total_bits()`), yielding `100.02548` to `100.26782` bits across the measured configurations. The draft accounts for evaluation-opening errors but has not been externally audited; composition and Fiat-Shamir proof obligations remain unproved. These estimates are not a certified full-protocol 100-bit security guarantee.
- **Scheme Constants**: Reed-Solomon code rates `1/2` and `1/4`; one evaluation point in $\mathbb{F}_{2^{128}}^k$ opened per run.
- **Point Sampling**: `ChaCha20Rng` with seed `0x4c4947455249544f XOR k`. Each run (including warmup) draws a fresh point from the sequence; the same `k` uses the same point sequence for both code rates. `point_field` and the decimal `point_seed` are recorded in the CSV.

## Measurement Methodology

- **Sampling**: 1 warmup followed by 3 measured samples per configuration; reported timings and proof sizes are **arithmetic means** of the 3 recorded samples (warmup discarded).
- **Timing Scope**: `Commit` measures initial commitment generation. `Open` reuses the commitment and **includes computing the claimed evaluation value** (`evaluation_ms`) as well as opening-proof generation (`open_proof_ms`). Point generation and proof serialization are excluded. `Prover` is the mean of each sample's `Commit + Open`, computed from the supplied per-run millisecond values before final rounding. `Verify` verifies the opening against the expected public commitment and extension-field claim. `Peak RSS` is the whole-process peak for that configuration, including warmup and setup, not a per-stage memory measurement.
- **Proof Size**: Mean `bincode` serialization size of `FinalizedLigeritoProof`, **including** the initial commitment (`initial_ligero_cm`). The separately supplied opening point and claimed evaluation value are not included. Sizes vary between runs, so `proof_bytes` is an arithmetic mean and may be fractional; `proof_kib` is calculated from the unrounded byte mean (`1 KiB = 1024 bytes`). Both are rounded to 3 decimal places.
- **Verification**: The benchmark verifies each measured opening and aborts if verification fails.

## Results

`k` denotes the number of multilinear variables (`2^k` `BinaryElem32` input evaluations). Time columns below use seconds for `Commit`, `Open`, and `Prover`, and milliseconds for `Verify`.

### Rate 1/2

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) |
|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.013 | 0.063 | 0.075 | 1.895 | 208.260 | 0.047 |
| 22 | 0.039 | 0.301 | 0.340 | 2.596 | 295.185 | 0.208 |
| 24 | 0.095 | 1.163 | 1.258 | 3.018 | 350.612 | 0.852 |
| 26 | 0.321 | 4.621 | 4.942 | 3.655 | 454.199 | 3.391 |
| 28 | 1.332 | 18.192 | 19.524 | 4.236 | 525.293 | 13.569 |
| 30 | 5.153 | 72.380 | 77.533 | 5.192 | 636.788 | 53.177 |

### Rate 1/4

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) |
|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.019 | 0.063 | 0.082 | 1.678 | 149.182 | 0.056 |
| 22 | 0.059 | 0.299 | 0.358 | 1.996 | 205.167 | 0.248 |
| 24 | 0.154 | 1.145 | 1.299 | 2.298 | 243.823 | 1.005 |
| 26 | 0.572 | 4.637 | 5.210 | 2.735 | 310.845 | 4.021 |
| 28 | 2.488 | 18.249 | 20.737 | 3.148 | 361.512 | 16.130 |
| 30 | 9.391 | 75.204 | 84.595 | 3.594 | 445.392 | 62.233 |

## Scheme-Specific Parameters & Notes

- The CSV records `evaluation_ms` and `open_proof_ms` separately. Their sum approximately equals `open_ms`; small differences reflect timer overhead and independent rounding. Do not add `evaluation_ms` to `open_ms` again.
- `queries` and `draft_opening_security_bits` per `(rate, k)` configuration are recorded in [`ligerito.csv`](ligerito.csv):
  - `rate = 1/2`: `queries` = 244, 245, 245, 246, 247, 248 for `k` = 20, 22, 24, 26, 28, 30.
  - `rate = 1/4`: `queries` = 149, 150, 150, 151, 152, 156 for `k` = 20, 22, 24, 26, 28, 30.
