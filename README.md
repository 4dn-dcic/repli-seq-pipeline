* This repo is a 4DN-DCIC fork of the original repli-seq pipeline Shart, without docker.

## what

This repository contains scripts in order to generate replication timing profiles from a set of raw reads from sequencing of either early- and late-replicating DNA, or from DNA extracted from cells sorted for S or G1 DNA content.

## how

#### define your input files

```bash
# download example data
wget -cbre robots=off -np -nH --cut-dirs=3 -A 'g*' http://www.bio.fsu.edu/~dvera/share/repliseq/
```

#### execute workflow step by step

```bash
# clip adapters from reads
clip $fastq

# align reads to genome
index=bwaIndex_hg38/genome
# paired-end
align_pe $fastq1 $fastq2 $index
# single-end
align_se $fastq $index

# check stats
samstats $bam

# filter bams by alignment quality and sort by position
filtersort $bam

# check stats
samstats $sbam

# remove duplicate reads
dedup $sbam

# calculate RPKM bedGraphs for each set of alignments
count $rbam

# filter windows with a low average RPKM
filter $bg

# calculate log2 ratios between early and late
log2ratio $fbg

# quantile-normalize replication timing profiles to the example reference bedGraph
normalize $l2r

# loess-smooth profiles using a 300kb span size
smooth -l 300000 $l2rn
```
