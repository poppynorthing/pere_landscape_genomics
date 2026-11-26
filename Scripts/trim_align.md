# Trimming and Alignment

### Trimming


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

#Trim sequence reads

fastp -i $RAW/"$file"_R1_001.fastq.gz -I $RAW/"$file"_R2_001.fastq.gz -o $TRIMDIR/"$file"_R1_001.fastq.gz -O $TRIMDIR/"$file"_R2_001.fastq.gz \
--adapter_sequence AGATCGGAAGAGCACACGTCTGAACTCCAGTCA \
--adapter_sequence_r2 AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
--trim_front1 10 --trim_front2 10 --cut_right --dedup -h $FASTPDIR/"$file".html -g -w 8
```

### Alignment


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

#Align trimmed reads to PERE reference genome

bwa mem -t 16 $GENOME $TRIMDIR/"$file"_R1_merged.fastq.gz $TRIMDIR/"$file"_R2_merged.fastq.gz > $ALIGNDIR/"$file".sam
samtools view -bS $ALIGNDIR/"$file".sam > $ALIGNDIR/"$file".bam
samtools sort $ALIGNDIR/"$file".bam -o $ALIGNDIR/"$file".bam
rm $ALIGNDIR/"$file".sam

# Run samtools flagstat
reads=`samtools flagstat $ALIGNDIR/"$file".bam -@ 24 | grep total | cut -f 1 -d' '`
mapped=`samtools flagstat $ALIGNDIR/"$file".bam -@ 24 | grep mapped | head -n 1 | cut -f 1 -d' '`
echo "${file},${reads},${mapped}" >> mapping_stats.csv
```

### References

