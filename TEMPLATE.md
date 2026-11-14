# <Scheme Name> Benchmark Results

Measurements of [<Implementation Name>](<repo-url>) for `k` (number of multilinear variables) from ... to .... (Note any local modifications if applicable.)

## Setup & Parameters

- **Hardware & Runtime**: e.g., AMD EPYC 9754, 503 GiB RAM, 32 threads
- **Field & Challenges**: e.g., Goldilocks (`2^64 - 2^32 + 1`) with cubic extension challenges
- **Hash / Transcript**: e.g., BLAKE3 Merkle hashing; Merlin Fiat–Shamir transcript
- **Security Target**: e.g., 100 bits (specify decoding regime / conditional assumptions / grinding bits if any)
- **Scheme Constants**: ...

## Measurement Methodology

- **Sampling**: Warmup + measured runs; specify **median** or **arithmetic mean**.
- **Timing Scope**: State what `Setup` / `Trim` / `Commit` / `Open` / `Prover` / `Verify` include or exclude.
- **Proof Size**: Reported in KiB (`1 KiB = 1024 bytes`) rounded to 3 decimal places; state whether it **includes** or **excludes** the commitment.
- **Verification & Missing Values**: Confirm all measured samples passed verification; `—` denotes an unmeasured metric.

## Results

`k` denotes the number of multilinear variables (`2^k` field elements).

| k | Commit (s) | Open (s) | Prover (s) | Verify (ms) | Proof (KiB) | Peak RSS (GiB) |
|---:|---:|---:|---:|---:|---:|---:|
| 20 | | | | | | |

## Scheme-Specific Parameters & Notes

- ...
