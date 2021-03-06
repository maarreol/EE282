cd /bio/maarreol

### Download fastq files:  
wget http://crick.bio.uci.edu/jiangs2/CSF1R/A1_S1_R1.fastq.gz  

_repeat for rest of fastq files_  

### Quality Check Fastq files:  
#!/bin/bash  
#$ -N Fastqc  
#$ -q pub8i, free128, free88i  
#$ -pe openmp 32  
#$ -R Y  
module load fastqc  
source /bio/maarreol  
fastqc *.gz  

### Unzip files:  
gunzip *.gz  

### Download mouse genome assembly:    
wget --timestamping 'ftp://hgdownload.cse.ucsc.edu/goldenPath/mm10/chromosomes/*'  

gunzip *.gz  

### Concatenate assembly:  

echo "$(ls chr*.fa | sort -V | grep -vP 'chr[^X|Y|\d]'; ls chr*.fa | sort -V | grep -vP 'chr[\d|X|Y]')" | xargs cat > mousegenome1.fa

### Index Genome:  

#! /bin/bash/  
#$ -N Star_genome_generation  
#$ -pe openmp 30  
#$ -q pub8i,free128,free88i  
module load STAR/2.5.2a  
STAR --runMode genomeGenerate --genomeDir /bio/maarreol --genomeFastaFiles /bio/maarreol/mousegenome1.fa --runThreadN 30

### Align Genome:  

#! /bin/bash/  
#$ -N Alignment_Star  
#$ -pe openmp 10  
#$ -q pub8i, free128, free88i  
module load STAR/2.5.2a  
STAR --genomeDir /bio/maarreol/ --runThreadN 24 --readFilesIn /bio/maarreol/A1_S1_R1.fastq /bio/maarreol/A1_S1_R2.fastq --outFileNamePrefix A1 --outSAMtype BAM Unsorted SortedByCoordinate

_replace necessary files and directory names in script for remaining files_ 

### Features Counts:  

wget ftp://ftp.ensembl.org/pub/release-94/gtf/mus_musculus/Mus_musculus.GRCm38.94.gtf.gz  

#! /bin/bash/  
#$ -N featurecounts  
#$ -pe openmp 10  
#$ -q pub8i, free128, free88i  

module load subread/1.5.0-p3  

featureCounts -p -t exon -g gene_id -a Mus_musculus.GRCm38.94.gtf -o counts.txt /bio/maarreol/A1Aligned.out.bam  /bio/maarreol/A2Aligned.out.bam /bio/maarreol/A3Aligned.out.bam /bio/maarreol/A4Aligned.out.bam /bio/maarreol/A5Aligned.out.bam  /bio/maarreol/A6Aligned.out.bam /bio/maarreol/B1Aligned.out.bam /bio/maarreol/B2Aligned.out.bam /bio/maarreol/B3Aligned.out.bam /bio/maarreol/B4Aligned.out.bam /bio/maarreol/B5Aligned.out.bam /bio/maarreol/B6Aligned.out.bam /bio/maarreol/C1Aligned.out.bam /bio/maarreol/C2Aligned.out.bam /bio/maarreol/C3Aligned.out.bam /bio/maarreol/C4Aligned.out.bam /bio/maarreol/C5Aligned.out.bam /bio/maarreol/C6Aligned.out.bam /bio/maarreol/D1Aligned.out.bam /bio/maarreol/D2Aligned.out.bam /bio/maarreol/D3Aligned.out.bam /bio/maarreol/D4Aligned.out.bam /bio/maarreol/D5Aligned.out.bam /bio/maarreol/D6Aligned.out.bam  

_Should give you two files "counts.txt" and "counts.txt.summary_  

Using counts.txt.summary we can look over the number of reads that fit under each assignment to see if there is any skew in any of the samples.  

### Quick Data Visualization of Feature Counts  

_Install any packages necessary in R_  

install.packages("dplyr")  
install.packages("tidyr")  
install.packages("ggplot2")  

x<-read.delim("counts.txt.summary",row.names=1) %>%  
t %>%  
as.data.frame %>%  
mutate(sample=gsub("Aligned.out.bam", "", rownames(.))) %>%  
select(sample, Assigned:Unassigned_NoFeatures) %>%  
gather(Assignment, ReadCounts, -sample) %>%  

x %>%  
ggplot (aes(Assignment, ReadCounts)) + geom_bar(stat="identity", aes(fill=sample),position="dodge")

![Rplot1](https://github.com/maarreol/EE282/blob/master/Rplot01.png)  

### Further Quality Control  

source("http://bioconductor.org/biocLite.R")  
biocLite("edgeR")  
library(edgeR)  

\# _import data into R_  
featurecounts<-read.delim("counts.txt", stringsAsFactors = FALSE)  

\# _Remove first two columns from seqdata_  
datacounts <- featurecounts[,-(1:2)]  

\# _Store GeneId as rownames_  
rownames(datacounts) <- featurecounts[,1]  

\# _Get CPM values_  
myCPM<-cpm(datacounts)  

\# _Find values greater than 1 in cpm counts_  
thresh <- myCPM > 1  

\# _Keep genes that have at least 2 values greater than 0.5 from cpm_
keep <- rowSums(thresh) >= 2  

\# _keep more highly expressed genes by subsetting rows_  
counts.keep <- datacounts[keep,]  
 
\# _create DEGlist_  
 DGE <- DGEList(counts.keep)  

\# _Plot Library Sizes_  
barplot(DGE$samples$lib.size,names=colnames(DGE),las=2, col="Green")  
title("Library Sizes")  

![LibrarySizes](https://github.com/maarreol/EE282/blob/master/librarysizes.png)  

\# _Get log2 counts per million_  
logDGE <- cpm(DGE,log=TRUE)  

\# _Check distributions of samples using boxplots with line corresponding to median logCPM_  
boxplot(logDGE, xlab="", ylab="Log2 counts per million",las=2, col="Green")  
abline(h=median(logDGE),col="blue")  
title("Boxplots of logCPMs")  

![BoxplotLog](https://github.com/maarreol/EE282/blob/master/boxplots.png)
