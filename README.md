Clinical Genomics 101 — Code & Resources Repository
Author: Andrius Baskys, MD, PhD
Repository: https://github.com/abaskys/ClinicalGenomics101-scripts
Purpose: Companion code and technical appendices for the Springer textbook Clinical Genomics 101.
This repository provides all command-line examples, QC procedures, and demonstration workflows referenced in the book. It is designed for clinicians, trainees, and analysts who want to reproduce the examples, run quality-control checks on their sequencing data, or learn the fundamental steps of variant analysis using open-source tools.
All scripts were selected for clarity, transparency, and reproducibility in clinical or academic settings.
📂 Repository Contents
1. QC Command Cheat Sheet (Appendix 4)
Essential short-form commands for quickly evaluating BAM files, read quality, coverage metrics, and FASTQ integrity.
samtools alignment QC
Depth and completeness checks
Insert-size estimation
FastQC and multiQC workflows
Reference build verification
These commands mirror the QC metrics highlighted in Chapter 3 and Appendix 1.
2. Trio Analysis Essentials (Appendix 3)
A minimal, clinically oriented set of steps for:
Confirming sample identity
Identifying de novo, recessive, and compound heterozygous variants
Prioritizing variants using phenotype-matching
Validating candidate findings in IGV
Applying inheritance logic to narrow differential diagnoses
Commands and workflow guidance are provided in structured, copy-and-paste form.
3. Variant Interpretation Tools & Examples
Links and usage notes for commonly used open-access resources:
gnomAD
ClinVar
dbNSFP
CADD
REVEL
SpliceAI
MetaRNN
UCSC Genome Browser
IGV
Where possible, command-line examples and minimal working examples (MWEs) are included.
4. Example Pipeline Outline
A compact, didactic overview of a transparent variant-analysis pipeline:
# 1. Alignment
bwa mem -t 8 hg38.fa R1.fastq.gz R2.fastq.gz > sample.sam

# 2. Convert + sort
samtools view -Sb sample.sam | samtools sort -o sample.bam

# 3. Index BAM
samtools index sample.bam

# 4. Variant calling
bcftools mpileup -Ou -f hg38.fa sample.bam | \
bcftools call -mv -Ov -o sample.vcf

# 5. Annotation
java -jar snpEff.jar GRCh38.86 sample.vcf > sample.ann.vcf
This pipeline is included for teaching purposes and mirrors the structure discussed in the textbook.
🛠 Installation (macOS / Linux)
All tools used in this repository are open-source and installable via Homebrew:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install samtools bedtools fastqc multiqc
Verify installation:
samtools --version
bedtools --version
fastqc --version
multiqc --version
🧪 Test Datasets
When possible, commands in this repository use generic filenames (sample.bam, sample.vcf).
Readers may substitute:
their own data, or
publicly available datasets (e.g., GIAB NA12878)
for practice and reproducibility.
📄 License
MIT License (recommended for educational and clinical training materials).
Feel free to modify or extend these scripts for academic or clinical use.
📬 Contact
For educational inquiries related to the textbook:
Andrius Baskys, MD, PhD
(Contact handled through academic channels.)
