# TSPA Benchmark (client-side, no networking)

This repository contains a small Rust crate (`tspa`) together with **manual, reproducible micro-benchmarks**
for evaluating **client-side costs** of a Threshold SPA-style protocol.

Benchmarks are intentionally implemented **without Criterion** and instead rely on:

- fixed warmup + sample counts,
- deterministic RNG seeding,
- explicit CLI parameters,

so results are stable, scriptable, and suitable for papers.

## What is benchmarked

### TSPA (client-side)
User-side costs for:
- registration
- authentication (client-side only)

All benchmarks measure **local computation only** (no networking).

## Implementation primitives

- Group: `ristretto255` (Curve25519/Ristretto)
- Hash: `BLAKE3`
- OPRF: DH-style OPRF over Ristretto (server key = scalar)
- Storage encryption: AES-256-CTR over 32 bytes (no auth)

## Repository layout

    Tspa_benchmark/
    ├── Cargo.toml
    ├── README.md
    ├── dockerfile
    └── src/
        ├── lib.rs
        ├── crypto.rs
        ├── protocols/
        │   └── tspa.rs
        └── bin/
            └── bench_tspa.rs

Each file under `src/bin/` is a standalone CLI benchmark binary.

## Output files

`bench_tspa` produces:

- `tspa_reg.dat`
- `tspa_auth.dat`

Format:

    nsp tsp samples warmup min_ns p50_ns p95_ns max_ns mean_ns stddev_ns

All times are in nanoseconds.

## Running locally

Build:

    cargo build --release

Run with defaults:

    cargo run --release --bin bench_tspa

Custom grid:

    cargo run --release --bin bench_tspa -- \
      --nsp 20,40,60,80,100 \
      --tsp-pct 20,40,60,80,100 \
      --warmup-iters 300 \
      --sample-size 2000 \
      --out-prefix tspa

## Running with Docker

Build:

    docker build -t tspa-benchmark .

Run:

    mkdir -p out
    docker run --rm -v "$(pwd)/out:/out" tspa-benchmark \
      --nsp 20,40,60,80,100 \
      --tsp-pct 20,40,60,80,100 \
      --out-prefix /out/tspa
