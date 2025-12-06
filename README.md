# ClinicalGenomics101-scripts
Clinical Genomics 101 book scripts
Appendix 4
QC Command Cheat Sheet for Clinicians and Analysts
Purpose	Command (copy and paste into Terminal)	Interpretation / Clinical Relevance
1. Basic alignment QC summary	samtools flagstat sample.bam	Reports total, mapped, and duplicate reads. >95 % mapped is expected; duplicates >20 % suggest over-amplification.
2. Count total aligned reads	samtools view -c sample.bam	Confirms total reads after alignment; helps check completeness before variant calling.
3. Mean coverage across genome or exome	`samtools depth -a sample.bam	awk '{sum+=$3} END {print "Average depth = "sum/NR"×"}'`
4. Fraction of bases ≥ 30×	`samtools depth -a sample.bam	awk '{if($3>=30) total++} {count++} END {print (total/count)*100" % bases ≥ 30×"}'`
5. Examine per-chromosome mapping	`samtools idxstats sample.bam	column -t`
6. View duplicate rate	`samtools flagstat sample.bam	grep 'duplicates'`
7. Confirm reference build used	`samtools view -H sample.bam	grep '@SQ'
8. Inspect insert-size distribution	`samtools stats sample.bam	grep 'insert size average'`
9. Run FastQC on raw reads	fastqc R1.fastq.gz R2.fastq.gz -o ./fastqc_results/	Generates per-read QC (quality, GC%, adapter content, duplication).
10. Combine multiple FastQC reports	multiqc ./fastqc_results/	Summarizes all samples into one HTML report—ideal for cohort-level QC review.
Here’s the QC Command Cheat Sheet rewritten as true bash commands — ready to copy and paste directly into a terminal. Each snippet begins with a comment (#) describing its purpose, following standard Linux shell style.
1. Basic alignment QC summary
samtools flagstat sample.bam

2. Count total aligned reads

samtools view -c sample.bam

3. Mean coverage across genome or exome

samtools depth -a sample.bam | awk '{sum+=$3} END {print "Average depth =",sum/NR,"x"}'

4. Fraction of bases ≥ 30×

samtools depth -a sample.bam | awk '{if($3>=30) total++} {count++} END {print (total/count)*100, "% bases ≥ 30x"}'

5. Per-chromosome mapping summary

samtools idxstats sample.bam | column -t

6. Duplicate read rate (after marking duplicates)

samtools flagstat sample.bam | grep 'duplicates'

7. Confirm reference genome build used

samtools view -H sample.bam | grep "@SQ" | head

8. Insert-size distribution

samtools stats sample.bam | grep 'insert size average'

9. Run FastQC on raw reads (paired-end)

fastqc R1.fastq.gz R2.fastq.gz -o ./fastqc_results/

10. Combine all FastQC reports into one summary multiqc ./fastqc_results/
Bonus Section: Running QC Commands and Calculating Per-Gene Coverage
1. How to Use These Commands
All the commands in this appendix can be executed directly from the Terminal on macOS or any Linux system. They assume you have a working .bam alignment file and optionally a .bed file listing your target genes or exons.
Required tools
•	samtools — for viewing, indexing, and analyzing BAM files
•	bedtools — for computing coverage over genomic intervals
•	fastqc — for raw FASTQ quality reports
•	multiqc — for summarizing multiple QC reports
•	awk — for processing text
Installation via Homebrew (recommended for macOS and Linux)
If you do not already have Homebrew installed:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
Then install the needed tools:
brew install samtools bedtools fastqc multiqc awk
To verify installation:
samtools --version
bedtools --version
fastqc --version
multiqc --version
All commands from this chapter can now be run directly from any terminal window.
2. Calculating Per-Gene Coverage
Step 1 – Get the BED file for your exome
When a laboratory performs whole-exome sequencing (WES), it uses a file called a BED file.
This file lists which parts of the genome were actually captured and sequenced. It usually has a name such as:
Agilent_SureSelect_V7_hg19_targets.bed
Ask the laboratory or sequencing provider for this file. You can say: “Please send me the BED file that defines the capture targets for this sample (the same one used during alignment and variant calling).” Once you receive it:
1.	Save it in the same folder as your BAM file.
2.	Open a Terminal window and check a few lines:
head targets.bed
You should see something like:
   chr1   11868   12227   DDX11L1
   chr1   12612   12721   DDX11L1
   chr1   13220   14409   WASH7P

The first column is the chromosome (e.g., chr1). The next two columns show where that region starts and ends. The last column is the gene name. Make sure the chromosome names match your BAM file. Check your BAM header with:
samtools view -H sample.bam | grep '@SQ' | head

If you see SN:chr1, your BED file must also begin with chr1. If you see SN:1, the BED file must use 1 (without “chr”). If they don’t match, fix the BED file:

add “chr” to each line (for macOS):

sed -i '' 's/^/chr/' targets.bed

or remove “chr” from each line:

sed -i '' 's/^chr//' targets.bed
Once your BED file matches the BAM, you are ready for Step 2 — to calculate average coverage and the percentage of bases above 30×.
Step 2 – Check sequencing depth (coverage)
Now that your BED file and BAM file are ready, you can measure how deeply the exome was sequenced. Coverage tells you how many times each base was read, and whether the test meets the usual clinical goal of ≥ 30× depth for most targeted bases.
2.1 – Average coverage across the exome
Copy and paste this command in the Terminal:
samtools depth -b targets.bed sample.bam | \
awk '{sum+=$3} END {print "Average coverage over targets =",sum/NR,"x"}'
What it does:
samtools depth checks how many reads overlap each base in your targets.
awk averages those numbers.

How to read it:
If you see something like Average coverage over targets = 120x, that means each base was read on average 120 times — good depth for clinical work.
2.2 – Percent of bases covered at ≥ 30×
This is the key number that most clinical labs report.
samtools depth -b targets.bed sample.bam | awk '{if($3>=30) t++} {n++} END {printf "%.2f%% of target bases ≥30x\n",(t/n)*100}'
What it does:
Counts how many bases reached at least 30× depth and divides that by the total number of target bases.

How to read it:
≥ 90 % → excellent, clinical-grade.
80 – 89 % → usable, but re-check low-coverage genes.
< 80 % → likely under-sequenced; data may be incomplete.

When you run these two commands, write down both numbers (average depth and percent ≥ 30×) — they are your quick “health check” for the exome.
3.1 – Make a list showing how well each gene was covered
Copy and paste this command into your Terminal:
bedtools coverage -a targets.bed -b sample.bam -hist | awk '$2!="all"{ if($5>=30){ ge[$4]+=$6 } te[$4]+=$6 } END{ for (g in te){ printf "%s\t%.2f%% ≥30x\n", g, (ge[g]/te[g])*100 } }' | sort -k2,2nr > gene_pct30x.txt
What it does:

Calculates what percentage of each gene’s bases were covered at 30× or higher and saves the results in a file called gene_pct30x.txt

Example of output inside the file:
APOE    100.00% ≥30x
CYP2D6   92.50% ≥30x
BRCA1   85.70% ≥30x
3.2 – Show only the genes that need attention (< 90 % ≥ 30×)
awk -F'\t' '{gsub(/% .*/,"",$2); if($2+0 < 90) print $0}' gene_pct30x.txt | head
What it does:
Filters the file to show only genes where less than 90% of the bases reached 30× coverage and prints the first few lines so you can quickly see any problem genes.
How to read it:

If no genes appear → excellent coverage.
If a few genes show values around 85–90% → normal; some exons are always tricky.
If many genes fall below 80% → the run may need to be repeated or verified.

Step 4 – How to interpret and report the results

Now that you’ve checked your coverage, you can interpret what the numbers mean in a clinical context. These results help you decide whether your data are strong enough for reliable variant interpretation.
4.1 – Key numbers to remember
d)	Average coverage tells you how deeply the targets were sequenced on average.

A good exome usually shows 80×–150× mean coverage.

d)	Percent of target bases ≥ 30× shows how much of the exome reached clinical depth.

≥ 90% means excellent quality.
80–89% is usually acceptable.
< 80% suggests that some regions were not well covered.

d)	Per-gene coverage report (from Step 3) pinpoints any problem areas.
4.2 – What to do with low-coverage genes
Do not panic. Some genes are naturally hard to capture because of high GC content, repeats, or pseudogenes. Check if they are clinically relevant. If a poorly covered gene is unrelated to your clinical question, you can simply note it. If it is important (e.g., BRCA1, CFTR, CYP2D6), consider confirming those areas by:
a)	Sanger sequencing,
b)	Targeted re-sequencing, or
c)	Re-running the sample with higher read depth.
4.3 – How to summarize in a report
You can include a short coverage summary in every case report. For example:
Sequencing Quality Clinical Summary:
a)	Whole-exome sequencing achieved an average coverage of 112×.
b)	95.4% of target bases were covered at ≥30× depth.
c)	Ten genes had <90% of bases at ≥30×, including ZIC5 and CYP2D6, which are known low-coverage regions.
d)	Low-coverage outliers (e.g., PMS2, CYP2D6, PKD1) often reflect homology issues, not lab error.
e)	Genes with <90 % of bases ≥30× may have unreliable variant calls and require Sanger confirmation or re-sequencing.
What if the target BED file is missing?
If you don’t have the vendor targets.bed (e.g., Agilent_SureSelect_V7_hg19_targets.bed), you can still check WES quality by using a generic coding-exon (CDS) BED. CDS = Coding DNA Sequence (the protein-coding parts of genes). Most exome kits primarily target CDS, so using a CDS BED is a good stand-in for your kit until you obtain the official BED.
Download the human hg19 (GRCh37) GTF (working link)
Download GENCODE v19 from ENCODE and unzip:

wget https://www.encodeproject.org/files/gencode.v19.annotation/@@download/gencode.v19.annotation.gtf.gz
gunzip gencode.v19.annotation.gtf.gz
Make BED files from the GTF (exons and CDS)
1) EXONS BED (all exons, with gene names) — ok for rough QC

awk 'BEGIN{FS=OFS="\t"}
  $3=="exon" && $1 ~ /^chr([0-9]{1,2}|[XYM])$/ {
    attr=$9; gene=attr;
    sub(/.*gene_name "/,"",gene); sub(/".*/,"",gene);
    if (gene==""||gene==" "){gene=attr; sub(/.*gene_id "/,"",gene); sub(/".*/,"",gene)}
    print $1,$4-1,$5,gene
}' gencode.v19.annotation.gtf | sort -k1,1 -k2,2n > gencode.v19.exons.hg19.bed

2) CDS BED (coding parts only) — best stand-in for kit targets

awk 'BEGIN{FS=OFS="\t"}
  $3=="CDS" && $1 ~ /^chr([0-9]{1,2}|[XYM])$/ {
    attr=$9; gene=attr;
    sub(/.*gene_name "/,"",gene); sub(/".*/,"",gene);
    if (gene==""||gene==" "){gene=attr; sub(/.*gene_id "/,"",gene); sub(/".*/,"",gene)}
    print $1,$4-1,$5,gene
}' gencode.v19.annotation.gtf | sort -k1,1 -k2,2n > gencode.v19.cds.hg19.bed
(If your BAM uses chromosomes without “chr”, remove it in the BED with 
sed -i '' 's/^chr//' file.bed on macOS or 
sed -i 's/^chr//' file.bed on Linux.)
Calculate the key metrics (overall and per-gene)
A) Overall % of CDS bases ≥30× (single, headline number)
samtools depth -b gencode.v19.cds.hg19.bed sample.bam | \
awk '{if($3>=30) t++} {n++} END {printf "%.2f%% of CDS bases ≥30x\n",(t/n)*100}'
B) Per-gene % ≥30× and a list of weak genes (pct30x.txt)
Create per-gene table: % of CDS bases covered ≥30x, saved to gene_pct30x.txt

bedtools coverage -a gencode.v19.cds.hg19.bed -b sample.bam -hist | \
awk '$2!="all"{ if($5>=30){ ge[$4]+=$6 } te[$4]+=$6 } \
     END{ for (g in te){ printf "%s\t%.2f%% ≥30x\n", g, (ge[g]/te[g])*100 } }' | sort -k2,2nr > gene_pct30x.txt

Show only genes needing attention (<90% ≥30x)

awk -F'\t' '{gsub(/% .*/,"",$2); if($2+0 < 90) print $0}' gene_pct30x.txt | head
How to read it (clinician-friendly):
Overall CDS ≥30×:

≥90–95% = clinical-grade;
80–89% = acceptable with caution;
<80% = under-sequenced.

gene_pct30x.txt: any gene <90% is a potential blind spot; note it in the report or confirm variants there by another method.

When you later receive the official kit BED

Replace gencode.v19.cds.hg19.bed with Agilent_SureSelect_V7_hg19_targets.bed in the same commands. The numbers will then exactly reflect your lab’s target design.

4.4 – Why this step matters
Even a perfect variant-calling algorithm cannot detect what was never sequenced. By checking these metrics yourself, you ensure that “no variant detected” really means no variant present, not variant in an unsequenced region.
4. Summary
With these commands, a clinician-scientist can:
Verify whether the data are diagnostically adequate,
Detect sample or capture inefficiencies early, and
Produce simple, reproducible QC summaries for reports.
<img width="936" height="1120" alt="image" src="https://github.com/user-attachments/assets/1f6af198-5f1d-46cf-b66c-c25a330d84e3" />
