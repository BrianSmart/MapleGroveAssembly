# Phased Genome Assembly and Evaluation Pipeline

This repository contains a series of PBS scripts designed to perform a complete, phased genome assembly from PacBio HiFi and Hi-C reads. The pipeline uses `hifiasm` for assembly and includes steps for data preparation, quality control with BUSCO and Merqury, and contact map generation with `pairtools` and `Pretext`.

The scripts are structured to be modular and easily reusable for new projects.

---

## Workflow Overview

The pipeline is broken down into sequential steps, with each step residing in its own numbered directory. Scripts should be executed in numerical order.

-   **01_hifiBamToFasta**: Converts the input PacBio HiFi BAM file into a gzipped FASTA format required by the assembler.
-   **02_03_meryl_genomescope**: Contains two scripts:
    -   `02_merylRawHifi.pbs`: Counts k-mers from the HiFi reads using Meryl.
    -   `03_genomescope.pbs`: Analyzes the k-mer spectrum with GenomeScope2 to estimate genome characteristics like size, heterozygosity, and ploidy.
-   **04_hifiasm**: Performs the main genome assembly using HiFi reads and scaffolds the contigs into chromosomes using Hi-C data.
-   **05_gfastats**: Converts the GFA (Graphical Fragment Assembly) output from hifiasm into standard FASTA files for the primary assembly and both haplotypes.
-   **06_buscoContigs**: Runs BUSCO (Benchmarking Universal Single-Copy Orthologs) to assess the completeness of the assemblies.
-   **07_merquryContigs**: Runs Merqury to evaluate assembly consensus quality (QV), completeness, and phasing based on k-mer content.
-   **08_contactMatrixContigs**: Aligns the Hi-C reads back to the final assemblies and generates contact maps to visually inspect the chromosome-scale scaffolding.

---

## How to Use This Pipeline

This pipeline is designed to be easily run on a new dataset with minimal changes.

### 1. Clone the Repository

Clone this repository to your HPC environment.

```bash
git clone <your_repository_url>
cd <repository_name>
```

### 2. Add Your Raw Data

Place your raw sequencing files into the `rawdata/` directory. You will need:
-   PacBio HiFi reads (in BAM format)
-   Paired-end Hi-C reads (e.g., `*_R1.fastq.gz` and `*_R2.fastq.gz`)

### 3. Configure and Run

For each script (`01` through `08`), you only need to edit the **CONFIGURATION** block at the top to match your new project.

**Example: Editing `04_hifiasm.pbs` for a new project named "Silver_Queen"**
```bash
# ====================================================================
# --- CONFIGURATION: EDIT THIS VARIABLE FOR A NEW PROJECT ---
# ====================================================================

# 1. Define the unique prefix for your current project.
PROJECT_NAME="Silver_Queen"

# ====================================================================

# You would also update the paths to the input HiFi and Hi-C reads
# in the "GLOBAL VARIABLES" section of the script if they are different.
```

After configuring a script, submit it to the scheduler:
```bash
qsub 01_hifiBamToFasta/01_hifiBamToFasta.pbs
```
Wait for the job to complete successfully, then configure and submit the next script in the sequence.

---

## Dependencies & Environment

This workflow was designed for a PBS-based HPC cluster and assumes the following:

-   **Conda:** A working Conda installation (`miniconda3` or `anaconda3`) is required to create the software environments.
-   **Software Modules:** The pipeline uses custom-built software modules for `hifiasm`, `gfastats`, and `meryl`. Instructions for creating these modules are commented in the header of each respective script.
-   **Conda Environments:** Several scripts require specific conda environments. The one-time setup commands are included in the header of each script (e.g., `conda create -n busco_env -c bioconda busco`).
