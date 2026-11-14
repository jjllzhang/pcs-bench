# Ligerito Benchmark Results

Measurements of a locally modified version of [ligerito](https://github.com/rotkonetworks/ligerito) adding `rate = 1/2` support and a reusable-commitment `commit` / single-Boolean-position `open` benchmark interface for `k` from 20 to 30.

## Setup & Parameters

- **Hardware & Runtime**: AMD EPYC 9754, 503 GiB RAM, 32 physical cores pinned to CPUs 0–31 (32 Rayon threads); optimized release build (`target-cpu=native`, LTO, single codegen unit)
- **Field & Challenges**: `BinaryElem32` ($\mathbb{F}_{2^{32}}$) base field with `BinaryElem128` ($\mathbb{F}_{2^{128}}$) extension challenges
- **Hash / Transcript**: BLAKE3 Merkle hashing (`ligerito-merkle`); SHA-256 Fiat–Shamir transcript (`FiatShamir::new_sha256`)
- **Security Target**: Project draft security estimate (`security_report().total_bits() >= 100.0`, yielding `100.03`–`100.27` bits across all measured configurations). Note: this draft security analysis has not been externally audited and does not fully cover the evaluation-opening soundness proof.
- **Scheme Constants**: Reed–Solomon code rates `1/2` and `1/4`; evaluation opening at a single Boolean hypercube position (`EvalClaim` at index `⌊2^k / 3⌋`).

## Measurement Methodology

- **Sampling**: 1 warmup followed by 3 measured samples per configuration; reported timings are **arithmetic means** across the 3 measured runs (warmup discarded).
- **Timing Scope**: `Commit` measures initial commitment generation; `Open` measures opening proof generation reusing the commitment; `Prover` is `Commit + Open` (mean) computed at full precision prior to rounding; `Verify` checks root consistency and verifies the opening proof against the claim. `Peak RSS` is process peak resident memory (including warmup and setup for that configuration).
- **Proof Size**: Reported in KiB (`1 KiB = 1024 bytes`) rounded to 3 decimal places from `bincode` serialization of `FinalizedLigeritoProof`; **includes** the initial commitment (`initial_ligero_cm`).
- **Verification**: Every measured sample passed verification.

## Results

`k` denotes the number of multilinear variables (`2^k` `BinaryElem32` elements).

### Rate 1/2

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) |
|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.013 | 0.076 | 0.090 | 1.848 | 207.479 | 0.05 |
| 22 | 0.039 | 0.372 | 0.411 | 2.566 | 294.081 | 0.24 |
| 24 | 0.093 | 1.425 | 1.518 | 2.985 | 350.987 | 0.98 |
| 26 | 0.322 | 5.993 | 6.315 | 3.705 | 452.605 | 3.89 |
| 28 | 1.277 | 24.055 | 25.332 | 4.304 | 525.605 | 15.57 |
| 30 | 5.161 | 98.101 | 103.262 | 5.007 | 634.652 | 61.17 |

### Rate 1/4

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) |
|---:|---:|---:|---:|---:|---:|---:|
| 20 | 0.020 | 0.076 | 0.095 | 1.390 | 148.619 | 0.07 |
| 22 | 0.058 | 0.365 | 0.423 | 2.026 | 204.854 | 0.28 |
| 24 | 0.153 | 1.429 | 1.582 | 2.255 | 245.573 | 1.13 |
| 26 | 0.566 | 5.984 | 6.550 | 2.755 | 311.137 | 4.54 |
| 28 | 2.483 | 24.078 | 26.561 | 3.125 | 362.168 | 18.13 |
| 30 | 9.506 | 98.918 | 108.424 | 3.620 | 444.496 | 70.24 |

## Scheme-Specific Parameters & Notes

- `queries` and `draft_security_bits` per `(rate, k)` configuration are recorded in [`ligerito.csv`](ligerito.csv):
  - `rate = 1/2`: `queries` = 244, 245, 245, 246, 247, 248 for `k` = 20, 22, 24, 26, 28, 30.
  - `rate = 1/4`: `queries` = 149, 150, 150, 151, 152, 156 for `k` = 20, 22, 24, 26, 28, 30.
- Unlike schemes that open an arbitrary extension-field point, `Open` here opens a single evaluation at a Boolean hypercube index (`EvalClaim`).
