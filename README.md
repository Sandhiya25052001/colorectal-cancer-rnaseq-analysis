# colorectal-cancer-rnaseq-analysis
End-to-end bulk RNA-seq analysis of colorectal cancer (GSE50760) including QC, trimming, HISAT2 alignment, featureCounts quantification, DESeq2 differential expression, and GSEA pathway enrichment.
# Bulk RNA-seq Pipeline and Differential Gene Expression Analysis

## Dataset

SRA Data: **GSE50760 (Colorectal Cancer)**

---

## Analysis Workflow

```
SRA Data (GSE50760)
        │
        ▼
Quality Control (FastQC)
        │
        ▼
Read Trimming (fastp)
        │
        ▼
Genome Alignment (HISAT2)
        │
        ▼
Gene Quantification (featureCounts)
        │
        ▼
Differential Expression (DESeq2)
        │
        ▼
Pathway Enrichment (GSEA – Hallmark)
```

---

## Project Structure

```
RNAseq-Colorectal-Cancer/
│
├── data/
│   └── sample_metadata.csv
│
├── results/
│   ├── fastqc/
│   ├── fastp_reports/
│   ├── alignment/
│   ├── counts/
│   ├── DEGs/
│   └── GSEA/
│
├── scripts/
│   ├── fastqc.sh
│   ├── fastp.sh
│   ├── hisat2_alignment.sh
│   ├── featurecounts.sh
│   └── deseq2_analysis.R
│
└── README.md
```

---

## Key Findings

* Identified **125 differentially expressed genes**
* Tumor and normal samples clearly separated in **PCA analysis**
* **MYC and E2F signaling pathways** were significantly enriched in the analysis, indicating altered regulation of cell-cycle related transcriptional programs in colorectal cancer
* Activation of **G2M checkpoint and DNA repair pathways**
* Evidence of **epithelial–mesenchymal transition (EMT)**

---
# Introduction

This project performs an end-to-end bulk RNA-seq analysis to identify transcriptional differences between primary colorectal cancer tissues (**Tumor**) and normal colon epithelium (**Control**). Public RNA-seq data from GEO accession **GSE50760** were analyzed using a standard RNA-seq workflow including quality control, read trimming, alignment, gene quantification, differential expression analysis, and pathway enrichment.

The goal of this analysis is to identify biological pathways altered in colorectal cancer samples compared to normal tissue.

---

# Dataset Description

The dataset **GSE50760** contains RNA-seq samples from colorectal cancer patients.

**Study objective (from GEO):**
Identify prognostic gene signatures associated with aggressiveness of colorectal cancer.

### Sequencing Details

| Property  | Value               |
| --------- | ------------------- |
| Platform  | Illumina HiSeq 2000 |
| Read type | Paired-end          |
| Library   | cDNA                |
| Organism  | Homo sapiens        |

---

# Samples Used in This Analysis

From the dataset, **six samples** were selected for differential expression analysis.

## Tumor (Primary Colorectal Cancer)

* SRR975551
* SRR975552
* SRR975553

## Normal Colon Tissue

* SRR975569
* SRR975571
* SRR975575

These samples represent **tumor tissue vs normal colon epithelium**.

---

# Quality Control

Quality control of raw sequencing reads was performed using **FastQC**.

The FastQC reports were examined to evaluate:

* Per base sequence quality
* GC content distribution
* Adapter contamination
* Sequence duplication levels
* Sequence length distribution

Six RNA-seq samples were analyzed:

### Tumor samples

* SRR975551
* SRR975552
* SRR975553

### Normal samples

* SRR975569
* SRR975571
* SRR975575

# FastQC Summary

## Basic Statistics

All samples showed consistent sequencing characteristics:

* Read length: **101 bp**
* GC content: **~50–51%**
* Sequencing platform: **Illumina HiSeq**
* Total reads per sample: **~27–41 million reads**

These values are typical for bulk RNA-seq experiments and indicate adequate sequencing depth.

---

## Per Base Sequence Quality

The per-base sequence quality plots show the distribution of Phred quality scores across read positions.

**Observations:**

* High base quality at the beginning of reads (**Q30–Q40**).
* Gradual decrease in quality toward the **3' end of reads**.
* Some bases in the last positions drop into the low-quality region.

This pattern is common in Illumina sequencing datasets due to signal decay over sequencing cycles.

Although FastQC flags this metric as **FAIL**, it is expected behavior for longer reads (100 bp) and does not necessarily indicate poor sequencing.

---

## GC Content Distribution

GC content across all samples is approximately **50–51%**, which is consistent with the expected GC content of the human transcriptome.

The GC distribution does not show abnormal peaks, indicating that the libraries are not strongly biased.

---

## Sequence Length Distribution

All reads have a uniform length of **101 bp**, which confirms that sequencing was performed using a fixed-length Illumina read protocol.

---

## Sequence Duplication Levels

Moderate duplication levels were observed.

This is common in RNA-seq datasets because highly expressed transcripts naturally generate many identical reads.

Therefore, duplication in RNA-seq does not necessarily indicate technical artifacts.

---

## Adapter Content

No significant adapter contamination was detected in the raw reads.

However, trimming was still performed to remove potential low-quality bases at read ends.

---

## Quality Control Conclusion

Overall, the sequencing reads show acceptable quality for RNA-seq analysis.

The main observation is a decrease in base quality toward the end of reads, which is typical for Illumina sequencing.

To improve alignment accuracy, reads were processed using **fastp** to trim low-quality bases and remove potential adapter sequences before downstream analysis.

---

# Read Trimming and Filtering

Raw sequencing reads were processed using **fastp (v0.23.4)** to remove low-quality bases and potential adapter contamination.

fastp performs:

* Quality filtering
* Adapter trimming
* Removal of reads containing excessive ambiguous bases (**N**)
* Read length filtering
* Generation of QC reports (**HTML and JSON**)

This step improves alignment accuracy and downstream expression quantification.

---

## fastp Command Used

```bash
fastp \
-i SRR975551_1.fastq \
-I SRR975551_2.fastq \
-o SRR975551_trimmed_1.fastq \
-O SRR975551_trimmed_2.fastq \
-h SRR975551_fastp.html \
-j SRR975551_fastp.json \
-w 8
```

The same procedure was applied to all six samples.

---

## fastp Summary Statistics

Across all samples:

* Original read length: **101 bp**
* Mean read length after trimming: **~100 bp**
* GC content: **~49–51%**
* Duplication rate: **~5–8%**

These values are typical for RNA-seq libraries.

---

## Quality Improvement After Trimming

**Before trimming:**

* Q20 bases: **~81–83%**
* Q30 bases: **~71–73%**

**After trimming:**

* Q20 bases: **~94–95%**
* Q30 bases: **~85–86%**

This indicates a substantial improvement in overall sequencing quality.

---

## Filtering Results

Across the six samples:

* **~75–78% of reads passed filtering**
* **~21–24% of reads were removed due to low quality**
* Reads containing excessive **N bases were negligible (~0.03%)**
* **No reads were discarded due to short length**

These results indicate that trimming successfully removed low-quality sequences while retaining the majority of usable reads.

---

## Insert Size Distribution

The insert size peak across samples ranged from **145–166 bp**, which is consistent with typical Illumina RNA-seq library preparation protocols.

---

## GC Content

GC content remained stable after filtering (**~50%**), suggesting that trimming did not introduce sequence bias.

---

## fastp Report Visualization

fastp generates interactive reports that can be opened in a browser.

Example:

```bash
firefox SRR975551_fastp.html
```

or

```bash
xdg-open SRR975551_fastp.html
```

These reports include:

* Quality score distribution
* Base content distribution
* Adapter trimming summary
* Insert size distribution
* Duplication rate

---

## Conclusion of Trimming Step

The **fastp preprocessing step** significantly improved sequencing quality by removing low-quality bases and potential sequencing artifacts.

The majority of reads passed filtering and retained high **Q20 and Q30 scores**, ensuring reliable downstream analysis including:

* Genome alignment
* Gene quantification
* Differential expression analysis

---

# Read Alignment

Trimmed reads were aligned to the **human reference genome (GRCh38)** using **HISAT2**, a fast and memory-efficient spliced aligner designed for RNA-seq data.

HISAT2 uses a **graph-based indexing strategy** that allows accurate mapping of reads spanning exon–exon junctions.

---

## HISAT2 Alignment Command

```bash
hisat2 -p 8 \
-x reference/genome_index \
-1 SRR975551_trimmed_1.fastq \
-2 SRR975551_trimmed_2.fastq \
-S SRR975551.sam
```

Convert SAM to sorted BAM:

```bash
samtools view -@ 8 -bS SRR975551.sam | samtools sort -@ 8 -o SRR975551_sorted.bam
```

Index BAM file:

```bash
samtools index SRR975551_sorted.bam
```

The same pipeline was applied to all six samples.

---

# Gene Quantification

Aligned reads were quantified using **featureCounts** to assign reads to genomic features (genes).

```bash
featureCounts \
-T 8 \
-p \
-t exon \
-g gene_id \
-a Homo_sapiens.GRCh_
```
# Differential Gene Expression Analysis (Tumor vs Normal)

## Overview

Differential gene expression analysis was performed using **DESeq2** to identify genes significantly altered between colorectal tumor samples (**SRR975551, SRR975552, SRR975553**) and normal colon samples (**SRR975569, SRR975571, SRR975575**).

After normalization and statistical modeling, genes were considered significant using the threshold:

* **Adjusted p-value (FDR) < 0.05**

A total of:

**125 significantly differentially expressed genes (DEGs)** were identified.

These genes represent transcriptional alterations associated with colorectal tumor development and cellular regulatory changes.

---

# PCA Analysis

![PCA Plot](results/DEGs/PCA_plot.png)

Your PCA plot shows:

* **PC1 explains 67% of total variance**
* **PC2 explains 17% of variance**

### Interpretation

The PCA plot demonstrates clear separation between tumor and normal samples, indicating that the global gene expression profiles differ substantially between the two groups.

**Key observations:**

* Tumor samples cluster together
* Normal samples cluster separately
* This confirms good biological signal and dataset quality

This separation validates that the tumor transcriptome is significantly altered compared to normal colon tissue.

---

# Volcano Plot Interpretation

![Volcano Plot](results/DEGs/volcano_plot.png)

The volcano plot highlights genes that are significantly upregulated or downregulated.

* **Red dots → Upregulated genes in tumor**
* **Blue dots → Downregulated genes in tumor**
* **Grey dots → Non-significant genes**

Genes with:

* **Adjusted p-value < 0.05**
* **|log2FoldChange| > 1**

were considered biologically meaningful.

The plot reveals a clear distribution of significant genes, indicating substantial transcriptional reprogramming in colorectal tumors.

---

# Top Downregulated Genes in Tumor

Examples from your dataset:

| Gene    | log2FoldChange | padj   |
| ------- | -------------- | ------ |
| DPEP1   | -5.78          | 3.5e-4 |
| GRIN2D  | -4.79          | 3.5e-4 |
| INHBA   | -5.10          | 3.5e-4 |
| CFAP74  | -6.95          | 3.5e-4 |
| SLC6A20 | -5.60          | 3.5e-4 |
| KRT80   | -5.13          | 3.5e-4 |

### Biological Meaning

These genes show reduced expression in tumor samples, suggesting suppression of pathways related to:

* epithelial function
* cell differentiation
* metabolic regulation

Downregulation of genes like **DPEP1** and **SLC family transporters** has been reported in gastrointestinal cancers and may reflect altered metabolic states in tumor cells.

---

# Top Upregulated Genes in Tumor

Examples:

| Gene      | log2FoldChange | padj   |
| --------- | -------------- | ------ |
| RMEL3     | 6.99           | 0.0016 |
| LINC03138 | 2.77           | 0.0023 |
| RBM24     | 3.15           | 0.0034 |
| HS3ST6    | 4.60           | 0.0053 |
| GLDN      | 3.99           | 0.0063 |
| MT1M      | 3.67           | 0.021  |

### Biological Meaning

Upregulated genes in tumors are associated with:

* transcriptional regulation
* RNA processing
* stress response
* cellular signaling

Genes such as **RBM24** and **MT1M** are known to participate in cellular proliferation and tumor progression.

---

# Heatmap Interpretation

![Heatmap](results/DEGs/heatmap_top_genes.png)

The heatmap shows the top differentially expressed genes across samples.

**Key patterns observed:**

* Tumor samples display distinct expression patterns compared to normal samples.
* Downregulated genes show lower expression in tumor samples.
* Upregulated genes show higher expression in tumor samples.

The clustering pattern clearly separates tumor and normal samples, reinforcing the robustness of the differential expression results.

---

# Hallmark Pathway Enrichment (GSEA)

![GSEA Enrichment Plot](results/GSEA/enrichment_plot.png)

Gene Set Enrichment Analysis using **MSigDB Hallmark pathways** identified several significantly enriched pathways.

### Significant pathways include:

| Pathway                                 | Interpretation                   |
| --------------------------------------- | -------------------------------- |
| MYC Targets V1 / V2                     | Increased oncogenic MYC activity |
| E2F Targets                             | Cell cycle progression           |
| G2M Checkpoint                          | Mitotic regulation               |
| DNA Repair                              | Genomic instability response     |
| Epithelial Mesenchymal Transition (EMT) | Tumor invasion and metastasis    |

---

# Biological Insight

The enrichment of **MYC** and **E2F** target pathways suggests activation of transcriptional programs that drive:

* cell proliferation
* DNA replication
* tumor growth

The **G2M checkpoint** and **DNA repair pathways** indicate increased cellular division and genomic stress in tumor cells.

Activation of the **epithelial–mesenchymal transition (EMT)** pathway suggests potential mechanisms underlying tumor invasion and metastasis.

---

# Example Enrichment Plot Interpretation

![E2F Enrichment](results/GSEA/E2F_enrichment.png)

The enrichment plot for **E2F Targets** demonstrates that genes associated with cell cycle regulation are predominantly located at the top of the ranked gene list.

This indicates that cell cycle regulatory genes are significantly enriched in tumor samples, consistent with the hallmark characteristic of uncontrolled cell proliferation in cancer.

---

# Biological Conclusion

The RNA-seq analysis reveals significant transcriptional differences between colorectal tumor and normal colon tissues.

### Key findings include:

1. Identification of **125 differentially expressed genes**.
2. Distinct clustering of tumor and normal samples in **PCA analysis**.
3. Significant upregulation of genes associated with tumor growth and transcriptional regulation.
4. Downregulation of genes related to normal epithelial function.
5. Enrichment of oncogenic pathways including:

* MYC signaling
* E2F transcriptional targets
* G2M checkpoint
* DNA repair
* epithelial-mesenchymal transition

These findings collectively highlight dysregulation of cell cycle control and oncogenic signaling pathways in colorectal cancer.

---

# Final Project Interpretation (One-line Summary)

This RNA-seq analysis identified **125 significant genes** and key oncogenic pathways (**MYC, E2F, G2M checkpoint, DNA repair, EMT**) that distinguish colorectal tumor samples from normal colon tissue, providing insights into transcriptional programs associated with colorectal cancer progression.

---

# Limitations

This study is based on a **small sample size (3 tumor vs 3 normal samples)**.

Although differential expression and pathway enrichment provide insights into transcriptional alterations, further validation using larger datasets or experimental approaches would strengthen the biological conclusions.
