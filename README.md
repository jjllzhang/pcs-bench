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
| Brakedown | `SdigCode3` (`lcpc`) | Goldilocks | BLAKE3 / Merlin | 100-bit | EPYC 9754 (32T) | 20–30 | [md](results/brakedown.md) · [csv](results/brakedown.csv) |
| BrakingBase | Tensor / BaseFold hybrid | Goldilocks | BLAKE2s-256 | 100-bit | EPYC 9754 (32T) | 20–28 | [md](results/brakingbase.md) · [csv](results/brakingbase.csv) |
| ERA | 2D algebraic (`agnostir`) | `secp256k1` scalar | BLAKE3-256 | 100-bit (conditional) | EPYC 9754 (32T) | 20–30 | [md](results/era.md) · [csv](results/era.csv) |
| HyperFold | `QA`, `RS` (`r∈{2,4,8}`, `c∈{2,4}`) | Goldilocks (ext3), M127 (ext2) | BLAKE3 | 100-bit (IOP + binding) | EPYC 9754 (32T) | 20–30 | [md](results/hyperfold.md) · [csv](results/hyperfold.csv) |
| Ligerito | RS (`rate=1/2, 1/4`, Boolean open) | `BinaryElem32` (ext `BinaryElem128`) | BLAKE3 / SHA-256 | 100-bit (draft, unaudited) | EPYC 9754 (32T) | 20–30 | [md](results/ligerito.md) · [csv](results/ligerito.csv) |
| WHIR | `evaluations=1`, `pow_bits_max=20` | Goldilocks (ext3) | BLAKE3 | 100-bit | EPYC 9754 (32T) | 20–30 | [md](results/whir.md) · [csv](results/whir.csv) |

## Conventions

- Each scheme lives in `results/<scheme>.md` (setup, methodology, summary tables, notes) and `results/<scheme>.csv` (machine-readable numbers).
- CSV timings use milliseconds (`*_ms`), proof sizes use `proof_kib` (`1 KiB = 1024 bytes`) and `proof_bytes` when available, and memory uses `peak_rss_gib`.
- Cross-scheme caveats (such as median vs. arithmetic mean, whether proof size includes the commitment, and field bit-width) are documented explicitly in each scheme's `.md` file.
- Use [`TEMPLATE.md`](TEMPLATE.md) when adding a new scheme.
