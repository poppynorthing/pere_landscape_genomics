# Processing poolseq samples using Grenedalf

We used [grenedalf](https://github.com/lczech/grenedalf) v0.6.3 to handle our poolseq analyses. We used our filtered and indexed bam files (see [trim_align.md](https://github.com/poppynorthing/pere_landscape_genomics/blob/main/Scripts/trim_align.md)) as our input.

First, we calculated mean depth per x using mosdepth vx (ref).

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
for bam in $ALIGNDIR/*.bam; do mosdepth -t 8 -n "$file" "$bam"; done
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
