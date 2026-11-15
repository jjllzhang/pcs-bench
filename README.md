# pcs-bench

Benchmark archive for code-based polynomial commitment schemes (PCS), structured as paired `<scheme>.md` + `<scheme>.csv` files under [`results/`](results/).

## Repository Structure

```text
pcs-bench/
├── README.md        # Index of benchmarked schemes
├── TEMPLATE.md      # Markdown template for adding new schemes
└── results/         # Paired <scheme>.md and <scheme>.csv files
```

## Benchmarked Schemes

| Scheme | Variant / Code | Field | Hash / Transcript | Security Target | Hardware | Range (`k`) | Data |
|---|---|---|---|---|---|---|---|
| BaseFold | `RS`, `QA` | Goldilocks (ext3), M127 (ext2) | BLAKE3 | 100-bit (conditional) | EPYC 9754 (32T) | 20–29 | [md](results/basefold.md) · [csv](results/basefold.csv) |
| Blaze (incomplete POC) | Binary RAA (3 accumulation passes); nested BaseFold | `GF(2)` bits (64-bit packed), internal `GF(2^128)` | Custom `Blake2s` using `blake2b_simd` | Not established (POC) | EPYC 9754 (32T) | 20-30 | [md](results/blaze.md) · [csv](results/blaze.csv) |
| Brakedown | `SdigCode3` (`lcpc`) | Goldilocks | BLAKE3 / Merlin | 100-bit | EPYC 9754 (32T) | 20–30 | [md](results/brakedown.md) · [csv](results/brakedown.csv) |
| BrakingBase | Tensor / BaseFold hybrid | Goldilocks | BLAKE2s-256 | 100-bit | EPYC 9754 (32T) | 20–28 | [md](results/brakingbase.md) · [csv](results/brakingbase.csv) |
| ERA | 2D algebraic (`agnostir`) | `secp256k1` scalar | BLAKE3-256 | 100-bit (conditional) | EPYC 9754 (32T) | 20–30 | [md](results/era.md) · [csv](results/era.csv) |
| FRI-Binius (`p3-binary-pcs`) | Cantor RS + BaseFold / ring-switch (`BinaryPcs`, `BooleanPcs`) | `GF(2^32/64/128)` / `GF(2)` bits | Keccak-256 | 94.5–102.4 bit algebraic | EPYC 9754 (32T) | 20–30 | [md](results/fri-binius.md) · [csv](results/fri-binius.csv) |
| HyperFold | `QA`, `RS` (`r∈{2,4,8}`, `c∈{2,4}`) | Goldilocks (ext3), M127 (ext2) | BLAKE3 | 100-bit (IOP + binding) | EPYC 9754 (32T) | 20–30 | [md](results/hyperfold.md) · [csv](results/hyperfold.csv) |
| Ligerito | RS (`rate=1/2, 1/4`) | `BinaryElem32` (ext `BinaryElem128`) | BLAKE3 / SHA-256 | 100-bit (draft, unaudited) | EPYC 9754 (32T) | 20–30 | [md](results/ligerito.md) · [csv](results/ligerito.csv) |
| WHIR | `evaluations=1`, `pow_bits_max=20` | Goldilocks (ext3) | BLAKE3 | 100-bit | EPYC 9754 (32T) | 20–30 | [md](results/whir.md) · [csv](results/whir.csv) |

> **Blaze is raw POC data, not a complete PCS benchmark.** Its Merkle construction binds only the first row and verification checks are incomplete. Outer calls returning `Ok` do not establish full verification; do not directly rank its `verify_ms` against complete PCS implementations.

## Conventions

- Each scheme lives in `results/<scheme>.md` (setup, methodology, summary tables, notes) and `results/<scheme>.csv` (machine-readable numbers).
- Blaze uses binary input over `GF(2)`: `2^k` field elements occupy `2^k` bits before packing. Equal `k` values across different fields do not imply equal input byte sizes.
- CSV timings use milliseconds (`*_ms`), proof sizes use `proof_kib` (`1 KiB = 1024 bytes`) and `proof_bytes` when available, and memory uses `peak_rss_gib`.
- Cross-scheme caveats (such as median vs. arithmetic mean, whether proof size includes the commitment, and field bit-width) are documented explicitly in each scheme's `.md` file.
- Use [`TEMPLATE.md`](TEMPLATE.md) when adding a new scheme.
