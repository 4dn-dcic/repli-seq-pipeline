* This repo is a 4DN-DCIC fork of the original repli-seq pipeline Shart, without docker.

## what

This repository contains scripts in order to generate replication timing profiles from a set of raw reads from sequencing of either early- and late-replicating DNA, or from DNA extracted from cells sorted for S or G1 DNA content.

## how

#### define your input files

```bash
# download example data
wget -cbre robots=off -np -nH --cut-dirs=3 -A 'g*' http://www.bio.fsu.edu/~dvera/share/repliseq/

# define early and late fastq files, here using sample data
E=$(ls *early*.fq.gz)
L=$(ls *late*.fq.gz)

index=bwaIndex_hg38/genome

```

#### execute workflow step by step

```bash
# clip adapters from reads
cfq=$(clip $E $L)

# align reads to genome
bam=$(align -i $index $cfq)
bstat=$(samstats $bam)

# filter bams by alignment quality and sort by position
sbam=$(filtersort $bam)
fbstat=$(samstats $sbam)

# remove duplicate reads
rbam=$(dedup $sbam)

# calculate RPKM bedGraphs for each set of alignments
bg=$(count $rbam)

# filter windows with a low average RPKM
fbg=$(filter $bg)

# calculate log2 ratios between early and late
l2r=$(log2ratio $fbg)

# quantile-normalize replication timing profiles to the example reference bedGraph
l2rn=$(normalize $l2r)

# loess-smooth profiles using a 300kb span size
l2rs=$(smooth 300000 $NTHREADS $l2rn)

```
#### or use pipes
```bash
clip $E $L | align -i $index | filtersort | dedup | count | filter | log2ratio | normalize | smooth
```
