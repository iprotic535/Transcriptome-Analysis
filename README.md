# Transcriptome-Analysis

# 🧬 RNA-seq Pipeline (Invertebrate, Paired-End)

This pipeline documents RNA-seq processing steps for invertebrate samples using FastQC, fastp, STAR, and featureCounts. All commands were run on Linux.

```bash
############################################
# 1. Quality control on raw reads
############################################
fastqc SRR34587932_1.fastq SRR34587932_2.fastq

############################################
# 2. Adapter trimming and quality filtering (fastp)
############################################
fastp \
  -i SRR34587932_1.fastq \
  -I SRR34587932_2.fastq \
  -o cleaned/SRR34587932_1.trim.fastq \
  -O cleaned/SRR34587932_2.trim.fastq \
  --detect_adapter_for_pe \
  --qualified_quality_phred 20 \
  --length_required 20 \
  --thread 16 \
  --html cleaned/SRR34587932_fastp.html \
  --json cleaned/SRR34587932_fastp.json

############################################
# 3. Quality control of trimmed reads
############################################
fastqc cleaned/SRR34587932_1.trim.fastq cleaned/SRR34587932_2.trim.fastq

############################################
# 4. STAR genome index generation
############################################
STAR \
  --runMode genomeGenerate \
  --runThreadN 8 \
  --genomeDir ref_STAR \
  --genomeFastaFiles ref/GCF_003254395.2_Amel_HAv3.1_genomic.fasta

############################################
# 5. Splice-aware alignment with STAR
############################################
STAR \
  --runThreadN 8 \
  --genomeDir ref_STAR \
  --readFilesIn cleaned/SRR34587932_1.trim.fastq cleaned/SRR34587932_2.trim.fastq \
  --outSAMtype BAM SortedByCoordinate \
  --outFileNamePrefix bam/SRR34587932_

############################################
# 6. Gene-level quantification using featureCounts
############################################
featureCounts \
  -Q 10 \
  -M \
  -s 0 \
  -T 8 \
  -p \
  -t exon \
  -g gene_id \
  -a ref/Amel_HAv3.1.fixed2.gtf \
  -o counts/Amel_SRR34587932_geneCounts.txt \
  bam/SRR34587932_Aligned.sortedByCoord.out.bam
```

### ✅ Notes:
- **Paired-end**: `-p` enabled
- **Unstranded library**: `-s 0`
- **Multi-mapping reads included**: `-M`
- **Minimum MAPQ**: `-Q 10`
- **Exon-based counting**: `-t exon`, grouped by `gene_id`

### 🧾 Software versions:
- FastQC v0.11.9
- fastp v0.23.2
- STAR v2.7.11b
- featureCounts v2.1.1

---

### 🔜 Next Steps:
- Import `geneCounts.txt` into DESeq2 or edgeR for differential expression analysis.
- Use alignment stats from `Log.final.out` to assess mapping quality.
