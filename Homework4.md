#### 1.  Calculate the following for all sequences ≤ 100kb and all sequences > 100kb:  

bioawk -c fastx '{if(length($seq) <= 100000){ print ">"$name"\n"$seq }}' dmel-all-chromosome-r6.24.fasta > 99kb.fasta  

bioawk -c fastx '{if(length($seq) > 100000){ print ">"$name"\n"$seq }}' dmel-all-chromosome-r6.24.fasta > 100kb.fasta  

faSize 99kb.fasta  

faSize 100kb.fasta  

1. Total Number of Nucleotides:  
Less than 100 kb:  
Greater than 100 kb:  

2. Total Number of Ns:  
Less than 100 kb:  
Greater than 100 kb:  

3. Total Number of Sequences:  
Less than 100 kb:  
Greater than 100 kb:

#### Plots of the following for the whole genome, for all sequences ≤ 100kb, and all sequences > 100kb:*  

1. Sequence Length Distribution  

samtools faidx 100kb.fasta
cut -f2 100kb.fasta.fai | Rscript -e 'data <- as.numeric (readLines ("stdin")); summary(data); hist(data)'  
  
_gives you plot as Rplots.pdf so change name before reusing command for other plots; use same command with 99kb.fasta and whole genome files_ 

2. Sequence GC% Distribution:  

bioawk -c fastx '{ print ">"$name; print gc($seq) }' 100kb.fasta > 100kbGC \  
cut -f2 100kbGC | Rscript -e 'data <- as.numeric (readLines ("stdin")); summary(data); hist(data)  

_gives you plot as Rplots.pdf so change name before reusing command for other plots; use same command with 99kb.fasta and whole genome files_  

3. Cumulative genome size sorted from largest to smallest sequences  

#### Assemble a genome from MinION reads  

1. Download reads  
wget https://hpc.oit.uci.edu/~solarese/ee282/iso1_onp_a2_1kb.fastq.gz  

2. Use minimap to overlap reads, use miniasm to construct an assembly and calculate N50 of assembly:  

\#!/bin/bash  
\#$ -N homework4  
\#$ -q free128,free88i,free72i  
\#$ -pe openmp 32  
\#$ -R Y  
module load jje/jjeutils  
n50 () {bioawk -c fastx ' { print length($seq); n=n+length($seq); } END { print n; } $
  | sort -rn   | gawk ' NR == 1 { n = $1 }; NR > 1 { ni = $1 + ni; } ni/n > 0.5 { print $1; $}

minimap=$(which minimap)  
miniasm=$(which miniasm)  
basedir=/pub/jje/ee282/$USER  
projname=nanopore_assembly  
basedir=$basedir/$projname  
raw=$basedir/$projname/data/raw  
processed=$basedir/$projname/data/processed  
figures=$basedir/$projname/output/figures  
reports=$basedir/$projname/output/reports  
createProject $projname $basedir  

ln -sf /bio/share/solarese/hw4/rawdata/iso1_onp_a2_1kb.fastq $raw/reads.fq  

$minimap -t 32 -Sw5 -L100 -m0 $raw/reads.fq{,} | gzip -1 > $processed/onp.paf.gz  
$miniasm -f $raw/reads.fq $processed/onp.paf.gz > $processed/reads.gfa  
awk ' $0 ~/^S/ { print ">" $2" \n" $3 } ' $processed/reads.gfa | tee >(n50 /dev/stdin > $reports/n50.txt) | fold -w 60 > $processed/unitigs.fa  


