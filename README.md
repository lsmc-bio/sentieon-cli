[![CI](https://github.com/Sentieon/sentieon-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/Sentieon/sentieon-cli/actions/workflows/ci.yml)

# Sentieon CLI

A command-line interface for the Sentieon software

## Installation from sdist (recommended)

Download the latest tar.gz file from the GitHub release page, https://github.com/sentieon/sentieon-cli/releases/ and install the package with pip:
```sh
curl -LO https://github.com/Sentieon/sentieon-cli/releases/download/v1.5.2/sentieon_cli-1.5.2.tar.gz
pip install sentieon_cli-1.5.2.tar.gz
```

## Installation with Poetry

Create a new python virtual environment for the project, if needed:
```
# Create a new venv, if needed
python3 -m venv /path/to/new/virtual/environment/sentieon_cli

# Activate the venv
source /path/to/new/virtual/environment/sentieon_cli/bin/activate
```

`sentieon-cli` uses [poetry](https://pypi.org/project/poetry/) for packaging and dependency management. Initially, you will need to install poetry:
```
pip install poetry
```

Clone this repository and cd into the root directory:
```
git clone https://github.com/sentieon/sentieon-cli.git
cd sentieon-cli
```

Use poetry to install the `sentieon-cli` into the virtual environment:
```
poetry install
```

You can then run commands from the virtual environment:
```
sentieon-cli ...
```

## Global arguments
The `sentieon-cli` supports the following global arguments:
- `--verbose`: verbose logging.
- `--debug`: debugging mode for more verbose logging.

## Supported pipelines
- [**DNAscope**](https://support.sentieon.com/docs/sentieon_cli/#dnascope) - DNAscope pipeline implementation for germline SNV and indel calling from short read data.
- [**DNAscope LongRead**](https://support.sentieon.com/docs/sentieon_cli/#dnascope-longread) - DNAscope LongRead pipeline implementations for germline SNV and indel calling from long read data.
- [**DNAscope Hybrid**](https://support.sentieon.com/docs/sentieon_cli/#dnascope-hybrid) - DNAscope short-long-hybrid pipeline.
- [**Sentieon Pangenome**](https://support.sentieon.com/docs/sentieon_cli/#sentieon-pangenome) - Sentieon pangenome alignment and variant calling. Our recommended pipeline for short-read small variant calling.

## Snakemake Workflow (DNAscope Hybrid)

A Snakemake version of the DNAscope Hybrid pipeline is available in `workflow/`. It mirrors the full DAG (22 rules) and provides **restart/resume** — if a run fails partway through, re-running the same command picks up where it left off.

### Prerequisites

In addition to the standard sentieon-cli prerequisites, you need:
- [Snakemake](https://snakemake.readthedocs.io/) ≥ 7
- `sentieon`, `samtools`, `bcftools`, `bedtools`, `mosdepth`, `multiqc`, `igzip` on `$PATH`

### Quick start

```sh
# 1. Clone / pull the branch
git fetch origin
git checkout feature/snakemake-hybrid-workflow

# 2. Copy the config template and fill in your paths
cp workflow/config.hybrid.yaml workflow/my_run.yaml
# Edit workflow/my_run.yaml — set reference, model_bundle, output_vcf,
# sr_r1_fastq / sr_aln, lr_aln / lr_align_input, etc.

# 3. Point temp to fast storage (recommended)
export TMPDIR=/dev/shm  SENTIEON_TMPDIR=/dev/shm

# 4. Dry-run to verify the DAG resolves
snakemake -s workflow/Snakefile --configfile workflow/my_run.yaml -j 1 -n

# 5. Execute
snakemake -s workflow/Snakefile --configfile workflow/my_run.yaml -j <cores>

# 6. If it fails partway, just re-run the same command — Snakemake resumes
snakemake -s workflow/Snakefile --configfile workflow/my_run.yaml -j <cores>
```

### Cluster execution (Slurm example)

```sh
snakemake -s workflow/Snakefile \
  --configfile workflow/my_run.yaml \
  --cluster "sbatch -p <partition> -c {threads} --mem=64G" \
  -j 8
```

## Snakemake Workflow (DNAscope LongRead — ONT / PacBio)

Covers the full DNAscope LongRead pipeline (~40 rules) for PacBio HiFi and Oxford Nanopore data. Includes multi-pass phasing-based variant calling, SV/CNV calling, and optional haploid-region handling.

### Quick start

```sh
# Copy the config template and fill in your paths
cp workflow/config.longread.yaml workflow/my_lr_run.yaml
# Edit workflow/my_lr_run.yaml — set reference, model_bundle, output_vcf, tech (HiFi/ONT),
# and either sample_input (BAMs) or fastq + readgroups.

# Point temp to fast storage (recommended)
export TMPDIR=/dev/shm  SENTIEON_TMPDIR=/dev/shm

# Dry-run
snakemake -s workflow/Snakefile.longread --configfile workflow/my_lr_run.yaml -j 1 -n

# Execute
snakemake -s workflow/Snakefile.longread --configfile workflow/my_lr_run.yaml -j <cores>

# Resume after failure
snakemake -s workflow/Snakefile.longread --configfile workflow/my_lr_run.yaml -j <cores>
```

### Cluster execution (Slurm example)

```sh
snakemake -s workflow/Snakefile.longread \
  --configfile workflow/my_lr_run.yaml \
  --cluster "sbatch -p <partition> -c {threads} --mem=64G" \
  -j 8
```

---

## Snakemake Workflow (Sentieon Pangenome — Illumina / Ultima)

Covers the Sentieon Pangenome pipeline (15 rules) — the recommended approach for short-read small variant calling. Uses pangenome graph alignment via `vg` and dual BWA + minimap2 alignment strategy.

### Prerequisites (in addition to above)

- `kmc` (patched version from [Sentieon/KMC](https://github.com/Sentieon/KMC/releases) required for BAM input mode)
- `vg` (variation graph toolkit)
- `igzip` (optional, falls back to `gzip`)

### Quick start

```sh
# Copy the config template and fill in your paths
cp workflow/config.pangenome.yaml workflow/my_pg_run.yaml
# Edit workflow/my_pg_run.yaml — set reference, model_bundle, output_vcf,
# gbz, hapl, pop_vcf (required), and either r1_fastq+r2_fastq+readgroup
# or sample_input (BAMs).

# Point temp to fast storage (recommended)
export TMPDIR=/dev/shm  SENTIEON_TMPDIR=/dev/shm

# Dry-run
snakemake -s workflow/Snakefile.pangenome --configfile workflow/my_pg_run.yaml -j 1 -n

# Execute
snakemake -s workflow/Snakefile.pangenome --configfile workflow/my_pg_run.yaml -j <cores>

# Resume after failure
snakemake -s workflow/Snakefile.pangenome --configfile workflow/my_pg_run.yaml -j <cores>
```

### Cluster execution (Slurm example)

```sh
snakemake -s workflow/Snakefile.pangenome \
  --configfile workflow/my_pg_run.yaml \
  --cluster "sbatch -p <partition> -c {threads} --mem=64G" \
  -j 8
```

---

## License
Unless otherwise indicated, files in this repository are licensed under a BSD 2-Clause License.
