# ESBMC-Arduino — Reproducibility CI

[![Reproducibility Check](../../actions/workflows/reproduce.yml/badge.svg)](../../actions/workflows/reproduce.yml)

This repository hosts the **GitHub Actions CI** that validates the reproducibility of the artifact published at:

> **Zenodo concept DOI:** [10.5281/zenodo.21014209](https://doi.org/10.5281/zenodo.21014209)
> *(concept DOI — always resolves to the latest version)*

The workflow runs on a **clean Ubuntu 22.04 runner** — no pre-installed ESBMC, CBMC, or Frama-C — and mirrors exactly what a reviewer would do following the artifact's instructions. It builds the verifier **from source** (no cached binaries), so a green run proves the artifact is reproducible end-to-end on a fresh machine.

## Quick Start — Run on Your Machine

### Prerequisites
- **Ubuntu 22.04** (recommended; macOS is partially supported — see notes in `setup.sh`)
- `git`, `sudo`, internet access
- The Zenodo tarball (`esbmc-arduino-artifact.tar.gz`) — download from [10.5281/zenodo.21014209](https://doi.org/10.5281/zenodo.21014209)

---

### 1. Download and extract the artifact

```bash
wget "https://zenodo.org/records/21034231/files/esbmc-arduino-artifact.tar.gz?download=1" \
     -O esbmc-arduino-artifact.tar.gz
tar xzf esbmc-arduino-artifact.tar.gz
cd esbmc-arduino-artifact
```

> You can also verify the integrity of the download against the published checksum:
> ```bash
> wget "https://zenodo.org/records/21034231/files/SHA256SUMS?download=1" -O SHA256SUMS
> sha256sum -c SHA256SUMS
> ```

### 2. Run setup (installs CBMC, Frama-C, MATIEC, and builds patched ESBMC from source)

```bash
bash setup.sh
```

> ⏱ **This takes ~10–20 minutes** on first run — ESBMC is built from source against LLVM 18.  
> Subsequent runs are cached. `sudo` is required for `apt` dependencies.  
> On completion, `env.sh` is written automatically and sourced by `reproduce.sh`.

### 3. Run the experiments

**Quick smoke test** (~minutes — validates environment and samples key results):
```bash
bash reproduce.sh --smoke
```

**Full reproduction** (all claims, full corpus):
```bash
bash reproduce.sh --full
```

### 4. Check results

Results are written to `./results/`:

```
results/
├── keystone.csv      — Claim 1: naive vs HAL annotator on real corpus
├── baseline.txt      — Claim 2: cross-tool (CBMC + Frama-C)
├── real_corpus.txt   — Claim 3: MATIEC-generated C + CBMC
└── matrix.txt        — Claim 4: controlled deployment matrix
```

---

### macOS note
macOS is partially supported. The setup script requires [Homebrew](https://brew.sh) and builds ESBMC against `llvm@18` (must match exactly). If the build fails, use Ubuntu 22.04 or provide a prebuilt ESBMC binary with the HAL annotator:

```bash
ESBMC=/path/to/prebuilt/esbmc bash setup.sh
```

---

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


