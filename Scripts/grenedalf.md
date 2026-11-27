# Processing poolseq samples using Grenedalf

We used [grenedalf](https://doi.org/10.1093/bioinformatics/btae508) v0.6.3 to handle our poolseq analyses (Czech et al. 2024). We used our filtered and indexed bam files (see [trim_align.md](https://github.com/poppynorthing/pere_landscape_genomics/blob/main/Scripts/trim_align.md)) as our input.

First, we calculated mean depth for each sample across the genome using [mosdepth](https://doi.org/10.1093/bioinformatics/btx699) v0.3.14 (Pederson and Quinlan 2017).

```
#!/bin/bash

#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=4gb
#SBATCH --cpus-per-task=16
#SBATCH --time=6:00:00
#SBATCH --array=1-28%14

file=$(cat ./file_lists/samples.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

export MAMBA_ROOT_PREFIX=/home/u5/pcnorthing/micromamba
eval "$(/opt/ohpc/pub/apps/micromamba/2.0.2-2/bin/micromamba shell hook --shell bash)"
micromamba activate grenedalf

ALIGNDIR=sequences/aligned/dedup

# Calculate depth 
do mosdepth -t 8 -n "$file" "$file".dedup.bam

# Summarize
grep total *.mosdepth.summary.txt >> depth.txt
```

### Calculating allele frequencies per locus in each pool

```
```

### Creating sync file for further analyses

```
```

### Calculating between-pool Fst

```
```

### Calculating pool diversity statistics

```
```

### References

Brent S Pedersen, Aaron R Quinlan, Mosdepth: quick coverage calculation for genomes and exomes, Bioinformatics, Volume 34, Issue 5, March 2018, Pages 867–868, https://doi.org/10.1093/bioinformatics/btx699

Lucas Czech, Jeffrey P Spence, Moisés Expósito-Alonso, grenedalf: population genetic statistics for the next generation of pool sequencing, Bioinformatics, Volume 40, Issue 8, August 2024, btae508, https://doi.org/10.1093/bioinformatics/btae508
