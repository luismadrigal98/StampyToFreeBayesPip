# StampyToFreeBayesPip: Genomic Alignment and Variant Calling Pipeline

## Overview

StampyToFreeBayesPip is a comprehensive genomic alignment and variant calling pipeline designed for high-performance computing environments. It automates the process from raw FASTQ files to variant calls, leveraging BWA and Stampy aligners for improved accuracy, and multiple variant callers (FreeBayes, bcftools, GATK) for sensitive variant detection. The pipeline is optimized for non-model organisms or divergent reference genomes.

## Features

- FASTQ preprocessing with quality control via fastp
- Efficient alignment using BWA with Stampy refinement
- Flexible duplicate handling strategies
- BAM file merging, sorting, and indexing
- Variant calling with multiple callers (FreeBayes, bcftools, GATK)
- Support for joint variant calling across samples
- Region-based parallelization for improved performance
- Automated SLURM job management
- Support for both paired-end and single-end sequencing data

## Installation

### Prerequisites

- Python 3.6+
- SLURM workload manager
- Python 2.7 (for Stampy and FreeBayes)
- The following bioinformatics tools:
  - fastp
  - BWA
  - Stampy
  - samtools
  - Picard
  - FreeBayes
  - bcftools
  - GATK (optional)

### Setup

1. Clone the repository:
```bash
git clone https://github.com/luismadrigal98/StampyToFreeBayesPip.git
cd StampyToFreeBayesPip
```

2. Ensure all dependencies are installed and in your PATH.

3. Make the pipeline scripts executable:
```bash
chmod +x QC.py StampyRunner.py BamProcessor.py VariantCaller.py
```

## Pipeline Components

### 1. QC.py - FASTQ Preprocessing

Handles preprocessing of FASTQ files using fastp, with options for splitting large files.

### 2. StampyRunner.py - Sequence Alignment

Performs alignment using BWA followed by Stampy for improved mapping quality.

### 3. BamProcessor.py - BAM File Processing

Processes aligned reads, including merging, duplicate handling, and indexing.

### 4. VariantCaller.py - Variant Calling and Merging

Calls variants using multiple variant callers and merges results across regions.

## Usage

### Quality Control and Preprocessing (QC.py)

```bash
# Run complete preprocessing workflow
./QC.py workflow \
  --input-dirs /path/to/fastq/dir1 /path/to/fastq/dir2 \
  --split-dirs /path/to/split/dir1 /path/to/split/dir2 \
  --fastp-dirs /path/to/output/dir1 /path/to/output/dir2 \
  --job-dirs /path/to/job/dir1 /path/to/job/dir2 \
  --submit \
  --fastp-control-param "-3 --complexity_threshold=20 --length_required=50"
```

### Alignment (StampyRunner.py)

```bash
# Run complete alignment workflow
./StampyRunner.py workflow \
  --input-dirs /path/to/preprocessed/dir1 /path/to/preprocessed/dir2 \
  --bwa-dirs /path/to/bwa/dir1 /path/to/bwa/dir2 \
  --bam-dirs /path/to/bam/dir1 /path/to/bam/dir2 \
  --stampy-dirs /path/to/stampy/dir1 /path/to/stampy/dir2 \
  --job-dirs /path/to/job/dir1 /path/to/job/dir2 \
  --reference /path/to/reference.fasta \
  --submit
```

### BAM Processing (BamProcessor.py)

```bash
# Run complete BAM processing workflow
./BamProcessor.py workflow \
  --input-dirs /path/to/stampy/dir1 /path/to/stampy/dir2 \
  --merged-dirs /path/to/merged/dir1 /path/to/merged/dir2 \
  --processed-dirs-base /path/to/processed/dir1 /path/to/processed/dir2 \
  --job-dirs /path/to/job/dir1 /path/to/job/dir2 \
  --duplicate-mode mark_remove \
  --submit
```

### Variant Calling (VariantCaller.py)

```bash
# Run standard variant calling
./VariantCaller.py call \
  --input-dirs /path/to/bam/dir1 /path/to/bam/dir2 \
  --output-dirs /path/to/variants/dir1 /path/to/variants/dir2 \
  --job-dirs /path/to/job/dir1 /path/to/job/dir2 \
  --reference /path/to/reference.fasta \
  --regions Chr_01 Chr_02 \
  --variant-caller freebayes \
  --submit

# Run joint variant calling with paired BAM directories
./VariantCaller.py joint-call \
  --input-dirs-1 /path/to/2012A/bam/dir1 /path/to/2012A/bam/dir2 \
  --input-dirs-2 /path/to/2012B/bam/dir1 /path/to/2012B/bam/dir2 \
  --output-dirs /path/to/variants/dir1 /path/to/variants/dir2 \
  --job-dirs /path/to/job/dir1 /path/to/job/dir2 \
  --reference /path/to/reference.fasta \
  --regions Chr_01 Chr_02 \
  --variant-caller bcftools \
  --submit

# Merge VCF files by chromosome
./VariantCaller.py merge \
  --input-dirs /path/to/variants/dir1 /path/to/variants/dir2 \
  --output-dirs /path/to/merged/dir1 /path/to/merged/dir2 \
  --job-dirs /path/to/job/dir1 /path/to/job/dir2 \
  --merge-mode by_chromosome \
  --submit
```

## Advanced Options

### Variant Calling Options

```
Commands:
  call         Run variant calling on individual BAM directories
  joint-call   Run joint variant calling with paired BAM directories
  merge        Merge VCF files by chromosome or globally

Common options:
  --reference      Path to reference FASTA file
  --regions        Genomic regions to process [default: all]
  --window-size    Window size for variant calling [default: 1000000]
  --variant-caller Variant caller to use [freebayes, bcftools, gatk]
  --threads        Number of threads per task [default: 2]
```

## Detailed Workflow

1. **Preprocessing (QC.py)**
   - Split large FASTQ files for parallel processing
   - Compress split files
   - Run fastp for quality control and adapter trimming

2. **Alignment (StampyRunner.py)**
   - Align reads to reference using BWA mem
   - Convert SAM output to sorted BAM
   - Refine alignments using Stampy

3. **BAM Processing (BamProcessor.py)**
   - Merge BAM files by sample
   - Process duplicates according to specified strategy
   - Add read groups and index BAM files

4. **Variant Calling (VariantCaller.py)**
   - Call variants using FreeBayes, bcftools, or GATK
   - Process by chromosomal regions for parallelization
   - Support joint calling across multiple samples
   - Merge region-specific VCF files into cohesive output

## Directory Structure

The pipeline expects directories to be organized as follows:

```
Project/
├── raw_data/            # Raw FASTQ files
├── split_fastq/         # Split FASTQ files (intermediate)
├── processed_fastq/     # Quality-controlled FASTQ files
├── bwa_sam/             # BWA alignment results (SAM format)
├── sorted_bam/          # Sorted BAM files
├── stampy_bam/          # Stampy-refined BAM files
├── merged_bam/          # Merged BAM files
├── final_bam/           # Final processed BAM files
├── variants/            # Variant calling results by region
├── merged_variants/     # Merged variant files
└── jobs/                # SLURM job scripts
```

## License

[MIT License]

## Contact

For questions or support, please contact:
- Author: Luis Javier Madrigal-Roca
- Email: l338m483@ku.edu