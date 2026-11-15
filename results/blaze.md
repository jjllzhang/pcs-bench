# Blaze Benchmark Results

Measurements of the original Blaze POC in [plonkish_basefold](https://github.com/hadasz/plonkish_basefold) for `k` from 20 to 30. A measurement harness was added without modifying protocol source.

## Setup & Parameters

- **Hardware & Runtime**: AMD EPYC 9754, 503 GiB RAM, 32 Rayon threads pinned to CPUs 0-31; release build with `nightly-2025-08-18` and `target-cpu=native`
- **Field & Challenges**: Binary input over $\mathbb{F}_2$, packed 64 bits per `Blazeu64`; internal folding, sumcheck, and BaseFold use `B128` ($\mathbb{F}_{2^{128}}$). `Blazeu64` is a packed-bit container, not a $\mathbb{F}_{2^{64}}$ field element.
- **Hash / Transcript**: Custom `Blake2s` hash type implemented using `blake2b_simd`, with `Blake2sTranscript` and `BlazeBlake2sTranscript`; not standard BLAKE2s despite the type name
- **Security Target**: Not established. These are incomplete POC measurements, not a certified 100-bit configuration or a reproduction of the paper's performance.
- **Scheme Constants**: 64 rows; outer binary repeat-accumulate (RAA-family) code at rate `1/4`, with 2004 queries; nested BaseFold at rate `1/2`, with 402 query repetitions and 2 chunks. Ordinary `open/verify`, not `faster_*`.

## Measurement Methodology

- **Sampling**: Each configuration runs in a separate process with 1 warmup followed by 3 measured samples; timings are **arithmetic means**. Setup, Trim, and input are reused within a configuration. Each sample uses two fresh transcripts; input and setup use system entropy without a fixed seed.
- **Timing Scope**: `Commit` includes encoding, Merkle construction, and writing the initial commitment. `Open` reuses the commitment and includes auxiliary proof generation, query messages, and transcript writes. Both include original in-function log printing. `Verify` measures the original verification call, excluding reader creation and proof-buffer copying. Input generation, Setup, Trim, and initial point sampling are excluded. Setup/Trim timings and peak RSS were not measured.
- **Proof Size**: Sum of the two transcript lengths for **one proof**, **including the initial commitment**. Public point, claimed value, and transport framing are excluded. Reported in KiB (`1 KiB = 1024 bytes`) rounded to 3 decimal places; exact byte counts are in [`blaze.csv`](blaze.csv).
- **Verification**: All 24 outer calls (including warmups) returned `Ok(())`, but the harness uses an `eval=0` placeholder and the original verification checks are incomplete. This does **not** establish successful full verification. In particular, `Verify` must not be directly ranked against complete PCS implementations.

## Results

`k` denotes an input of `2^k` elements over $\mathbb{F}_2$, hence `2^k` bits before packing. All timing columns below use milliseconds.

| k | Commit (ms) | Open (ms) | Verify (ms)* | Proof (KiB) |
|---:|---:|---:|---:|---:|
| 20 | 1.424 | 58.017 | 7.248 | 7,761.078 |
| 22 | 1.631 | 98.191 | 11.266 | 9,318.609 |
| 24 | 3.039 | 146.370 | 7.476 | 10,976.641 |
| 26 | 6.341 | 262.360 | 9.421 | 12,735.172 |
| 28 | 61.256 | 631.230 | 8.097 | 14,594.203 |
| 30 | 386.162 | 2,390.033 | 10.321 | 16,553.734 |

\* Incomplete POC verification only.

## Scheme-Specific Parameters & Notes

- With 64 packed rows, each row contains `2^(k - 12)` words, and input size is `2^(k - 3)` bytes (128 KiB to 128 MiB). This sweep fixes the row count and increases row length. Equal `k` across different fields does not imply equal input byte size.
- The measured `commit_and_write -> encode_bits_ser` path performs three accumulation passes. Its `repeat_interleave` returns the pre-interleaving repetition. Nested BaseFold uses `random` code parameters from `setup/trim`, despite local `binary_rs` types in `open/verify`; it is not reported as a consistent additive-RS configuration.
- **POC limitations**: The Merkle root binds only the first row; Merkle path consistency assertions are disabled; nested BaseFold verification errors are ignored; evaluation-claim and sumcheck checks are incomplete. Thus all timings and proof sizes describe the original incomplete implementation, not a complete valid Blaze PCS.
- Matching original measurement harness/binary copies are unavailable. The original raw samples and environment files are not included in this archive.
