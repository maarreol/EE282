cd /bio/maarreol

Download fastq files:  
wget http://crick.bio.uci.edu/jiangs2/CSF1R/A1_S1_R1.fastq.gz  

_repeat for rest of fastq files_  

Quality Check Fastq files:  
#!/bin/bash  
#$ -N Fastqc  
#$ -q free128,abio128,bio,abio  
#$ -pe openmp 32  
#$ -R Y  
module load fastqc  
source /bio/maarreol  
fastqc *.gz  

Unzip files:  
gunzip *.gz  

Download mouse genome assembly:    
wget --timestamping 'ftp://hgdownload.cse.ucsc.edu/goldenPath/mm10/chromosomes/*'  

gunzip *.gz  

Concatenate assembly:  

echo "$(ls chr*.fa | sort -V | grep -vP 'chr[^X|Y|\d]'; ls chr*.fa | sort -V | grep -vP 'chr[\d|X|Y]')" | xargs cat > mousegenome1.fa

Index Genome:  

#! /bin/bash/  
#$ -N Star_genome_generation  
#$ -pe openmp 30  
#$ -q bio,epyc,pub8i,free128,free88i  
module load STAR/2.5.2a  
STAR --runMode genomeGenerate --genomeDir /bio/maarreol --genomeFastaFiles /bio/maarreol/mousegenome1.fa --runThreadN 30

Align Genome:  

#! /bin/bash/  
#$ -N Star_alignment_A1  
#$ -pe openmp 10  
#$ -q bio,epyc,pub8i  
module load STAR/2.5.2a  
STAR --genomeDir /bio/maarreol/ --runThreadN 24 --readFilesIn /bio/maarreol/A1_S1_R1.fastq /bio/maarreol/A1_S1_R2.fastq --outFileNamePrefix A1 --outSAMtype BAM Unsorted SortedByCoordinate

_replace necessary files and directory names in script for remaining files_ 

Features Counts:  

wget ftp://ftp.ensembl.org/pub/release-94/gtf/mus_musculus/Mus_musculus.GRCm38.94.gtf.gz  



