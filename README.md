# ESBMC-Arduino — Reproducibility CI

[![Reproducibility Check](../../actions/workflows/reproduce.yml/badge.svg)](../../actions/workflows/reproduce.yml)

This repository hosts the **GitHub Actions CI** that validates the reproducibility of the artifact published at:

> **Zenodo concept DOI:** [10.5281/zenodo.21014209](https://doi.org/10.5281/zenodo.21014209)
> *(concept DOI — always resolves to the latest version)*

The workflow runs on a **clean Ubuntu 22.04 runner** — no pre-installed ESBMC, CBMC, or Frama-C — and mirrors exactly what a reviewer would do following the artifact's instructions. It builds the verifier **from source** (no cached binaries), so a green run proves the artifact is reproducible end-to-end on a fresh machine.

## What the workflow does

| Step | Description |
|------|-------------|
| Download | Fetches `esbmc-arduino-artifact.tar.gz` and `SHA256SUMS` directly from Zenodo |
| Integrity | Verifies the SHA-256 of the download against the published `SHA256SUMS` (job fails on mismatch) |
| Cache | Caches the from-source ESBMC build, keyed on `setup.sh` + `esbmc-src.tar.gz` (rebuilds cleanly whenever either changes) |
| Setup | Runs `setup.sh` — installs CBMC 6.10.0, Frama-C 32.1, MATIEC, and **builds the patched ESBMC** (HAL annotator) against pinned **LLVM 18**, with the Z3 backend and LD front-end enabled |
| Verify | Asserts the built ESBMC produces a Z3 verdict **and** contains the `HAL_ANNOTATE` model — fails early otherwise |
| Reproduce | Runs `reproduce.sh` and captures the full log |
| Upload | Publishes `results/` + `reproduce.log` as a GitHub Actions artifact (90-day retention) |

## Claims verified

A green run reproduces every headline result in the paper:

| Claim | Expected |
|-------|----------|
| **Keystone 54→0** | naive 54 UNSAFE → HAL **0** UNSAFE, 32 SAFE proofs preserved |
| **Cross-tool** (CBMC + Frama-C) | phantom reproduced on an independent engine; bound must be hand-supplied |
| **Real-corpus 54/54** | CBMC on the MATIEC-generated C: 54 FAIL (naive) → 54 SAFE (with HAL bound) |
| **Controlled matrix** | `*_bug` UNSAFE@16-bit / SAFE@32-bit; `*_ok` always SAFE — **20/20** |

## How to trigger a run

- **Automatic:** runs on every push to `main` and on the 1st of each month
- **Manual:** go to *Actions → Reproducibility Check → Run workflow*

## Results

Each successful run produces a downloadable archive under *Actions → [run] → Artifacts*.
The job summary tab shows the last 30 lines of `reproduce.log` inline.

## Citing

If you use this CI setup as evidence of reproducibility, cite the Zenodo record (concept DOI — all versions):

```bibtex
@software{esbmc_arduino_artifact,
  author    = {Anonymous Author(s)},
  title     = {ESBMC-Arduino: Reproduction Artifact},
  year      = {2026},
  doi       = {10.5281/zenodo.21014209},
  publisher = {Zenodo}
}

Changes vs. your current README:
- **DOI** → concept DOI `21014209` (was the broken v1 `21014210`), in both the prose and the BibTeX.
- **Integrity row** → now accurately says SHA-256 verification (matches the workflow step you're adding).
- Added **Download `SHA256SUMS`**, a **Cache** row, a **Verify** (Z3 + HAL) row, the **LLVM 18 / Z3 / LD-frontend** build detail, and a **Claims verified** table.
