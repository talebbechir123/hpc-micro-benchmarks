# HPC Micro-Benchmarks — DGEMM, Dot Product, Reduction

Performance study of three fundamental numerical kernels in C — dense matrix multiplication (DGEMM), dot product and sum reduction — comparing loop orderings, manual unrolling, compilers, optimisation levels and a BLAS reference, under controlled measurement conditions.

Coursework for *OBHPC* (TD2, *Programmation C et mesures de performances*). Assignment: [`TD2.pdf`](TD2.pdf).

## Kernels

| Benchmark | Variants |
|---|---|
| `dgemm/` | `IJK` (naive), `IKJ` (loop interchange), `IEX` (interchange + invariant extraction), `UNROLL` (×4), `UNROLLX8`, `CBLAS` (`cblas_dgemm`) |
| `dotprod/` | baseline, unrolled ×8 |
| `reduc/` | baseline, unrolled ×8 |

Each kernel runs `r` times on `n`-element inputs; the harness reports min / max / mean, relative standard deviation and memory bandwidth (MiB/s), one CSV per compiler and optimisation level.

## Methodology

To get stable, reproducible numbers:

- laptop on mains power, CPU frequency fixed with `cpupower`
- process pinned to one core with `taskset`
- target architecture recorded in [`sys-info/`](sys-info/) (Intel Core i5-6300U, Skylake, AVX2; L1/L2/L3 cache parameters)
- builds with **gcc** and **clang** at several optimisation levels, one result folder per compiler and level (`gcc-v/`, `clang-v/`)

## Sample result

DGEMM, gcc, `n = 100`, 50 repetitions (`dgemm/gcc-v/03`):

| Variant | Bandwidth (MiB/s) | Speed-up vs IJK |
|---|---:|---:|
| IJK | 71.6 | 1.0× |
| IKJ | 219.9 | 3.1× |
| UNROLL | 235.8 | 3.3× |
| UNROLLX8 | 241.0 | 3.4× |
| IEX | 289.6 | 4.0× |
| CBLAS | 292.1 | 4.1× |

Loop order alone (IJK → IKJ, unit-stride access on `B`) gives a 3× gain, hoisting `a[i][k]` out of the inner loop adds another 30%; the best hand-written variant reaches the BLAS reference at this size. All stability and gcc/clang comparison charts are in [`plots/`](plots/).

## Build and run

Requires a C compiler and CBLAS (`libcblas` / OpenBLAS; Intel MKL for `icc`/`icx`).

```bash
cd dgemm
make CC=gcc               # or CC=clang, CC=icx, CC=icc
taskset -c 2 ./dgemm 100 50      # n, repetitions
```

`dotprod/` and `reduc/` work the same way. Plots are generated with the gnuplot scripts (`plot*.gp`) in each folder.

## Author

**Ahmed Taleb Bechir**
[atbechir.com](https://atbechir.com) · [LinkedIn](https://www.linkedin.com/in/ahmed-taleb-bechir-97a7301a4) · [GitHub](https://github.com/talebbechir123)
