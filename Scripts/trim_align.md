# Trimming and Alignment

### Trimming
Raw PE sequence reads were trimmed using [fastp](https://github.com/opengene/fastp) v0.23.4 (Chen 2025). We removed illumina adapter sequences, hard clipped the first 10 bases to remove barcodes, removed low-quality 3' reads (--cut_right), and removed PCR duplicate reads (--dedup).

```
#!/bin/bash

#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=4gb
#SBATCH --cpus-per-task=8
#SBATCH --time=2:00:00
#SBATCH --array=1-224%25

file=$(cat ./file_lists/raw_file_prefixes.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml fastp

RAW=sequences/raw
TRIMDIR=sequences/trimmed
FASTPDIR=sequences/trimmed/fastp_report

# Trim sequence reads

fastp -i $RAW/"$file"_R1_001.fastq.gz -I $RAW/"$file"_R2_001.fastq.gz -o $TRIMDIR/"$file"_R1_001.fastq.gz -O $TRIMDIR/"$file"_R2_001.fastq.gz \
--adapter_sequence AGATCGGAAGAGCACACGTCTGAACTCCAGTCA \
--adapter_sequence_r2 AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
--trim_front1 10 --trim_front2 10 --cut_right --dedup -h $FASTPDIR/"$file".html -g -w 8
```

### Alignment
Trimmed reads were aligned to the chromosome contigs from the [<i>Pectocarya recurvata</i> reference genome](https://bsapubs.onlinelibrary.wiley.com/doi/10.1002/aps3.70008) (Northing et al. 2025) using [BWA](https://github.com/lh3/bwa) mem v0.7.18 (Li 2013) with default settings.

```
 #!/bin/bash

#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=4gb
#SBATCH --cpus-per-task=16
#SBATCH --time=2-12:00:00
#SBATCH --array=1-28%28

file=$(cat ./file_lists/samples.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml bwa
ml samtools

GENOME=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa
TRIMDIR=sequences/trimmed
ALIGNDIR=sequences/aligned

# Align trimmed reads to PERE reference genome

bwa mem -t 16 $GENOME $TRIMDIR/"$file"_R1_merged.fastq.gz $TRIMDIR/"$file"_R2_merged.fastq.gz > $ALIGNDIR/"$file".sam
samtools view -bS $ALIGNDIR/"$file".sam > $ALIGNDIR/"$file".bam
samtools sort $ALIGNDIR/"$file".bam -o $ALIGNDIR/"$file".bam
rm $ALIGNDIR/"$file".sam

# Generate read mapping statistics
reads=`samtools flagstat $ALIGNDIR/"$file".bam -@ 24 | grep total | cut -f 1 -d' '`
mapped=`samtools flagstat $ALIGNDIR/"$file".bam -@ 24 | grep mapped | head -n 1 | cut -f 1 -d' '`
echo "${file},${reads},${mapped}" >> mapping_stats.csv
```

### Mark Duplicate Reads
PCR and optical duplicates were removed using [picard](https://github.com/broadinstitute/picard) v2.23.4 (Picard Toolkit 2019). Make sure that picard has enough memory; I had to set mem-per-cpu to 16
to get this to run.

```
#!/bin/bash

#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=16gb
#SBATCH --cpus-per-task=2
#SBATCH --time=1-00:00:00
#SBATCH --array=1-28%28

file=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml picard

BAMDIR=sequences/aligned

# Mark duplicates (pcr + optical) w/ picard

picard MarkDuplicates I=$BAMDIR/"$file"_merged.rg.bam O=$BAMDIR/"$file".markedDups.bam M="$file".dupMetrics.txt

```

### References

Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, Richard Durbin, 1000 Genome Project Data Processing Subgroup, The Sequence Alignment/Map format and SAMtools, Bioinformatics, Volume 25, Issue 16, August 2009, Pages 2078–2079, https://doi.org/10.1093/bioinformatics/btp352

Li H. (2013) Aligning sequence reads, clone sequences and assembly contigs with BWA-MEM. arXiv:1303.3997v2 [q-bio.GN].

Northing PC, Pelosi JA, Venable DL, Dlugosch KM. Chromosome-scale reference genome of Pectocarya recurvata, the species with the smallest reported genome size in Boraginaceae. Appl Plant Sci. 2025 May 21;13(3):e70008. doi: 10.1002/aps3.70008.

Picard Toolkit. 2019. Broad Institute, GitHub Repository. https://broadinstitute.github.io/picard/; Broad Institute
