#### 1.  Calculate the following for all sequences ≤ 100kb and all sequences > 100kb:  

bioawk -c fastx '{if(length($seq) <= 100000){ print ">"$name"\n"$seq }}' dmel-all-chromosome-r6.24.fasta > 99kb.fasta  

bioawk -c fastx '{if(length($seq) > 100000){ print ">"$name"\n"$seq }}' dmel-all-chromosome-r6.24.fasta > 100kb.fasta  

faSize 99kb.fasta  

faSize 100kb.fasta  

#### 1. Total Number of Nucleotides:  
Less than or equal to 100 kb: 6178042  
Greater than 100 kb: 137547960  

#### 2. Total Number of Ns:  
Less than 100 kb: 662593  
Greater than 100 kb: 490385  

#### 3. Total Number of Sequences:  
Less than 100 kb: 1863  
Greater than 100 kb: 7

#### Plots of the following for the whole genome, for all sequences ≤ 100kb, and all sequences > 100kb:*  

#### 1. Sequence Length Distribution  

samtools faidx 100kb.fasta
cut -f2 100kb.fasta.fai | Rscript -e 'data <- as.numeric (readLines ("stdin")); summary(data); hist(data)'  
  
_gives you plot as Rplots.pdf so change name before reusing command for other plots; use same command with 99kb.fasta and whole genome files_ 

#### 2. Sequence GC% Distribution:  

bioawk -c fastx '{ print ">"$name; print gc($seq) }' 100kb.fasta > 100kbGC \  
cut -f2 100kbGC | Rscript -e 'data <- as.numeric (readLines ("stdin")); summary(data); hist(data)  

_gives you plot as Rplots.pdf so change name before reusing command for other plots; use same command with 99kb.fasta and whole genome files_  

#### 3. Cumulative genome size sorted from largest to smallest sequences  

mkfifo 100kb_fifo

bioawk -c fastx 'length($seq) > 100000 { print length($seq) }' *fasta \
| sort -rn | awk ' BEGIN { print "Assembly\tLength\nSeqLength>100kb\t0" } { print "SeqLength>100kb\t" $1 } ' \
\> 100kb_fifo & 

plotCDF2 100kb_fifo 100kb.png  
display 100kb.png

_repeat steps for other two graphs replacing necessary names and files_

#### Assemble a genome from MinION reads  

#### 1. Download reads  
wget https://hpc.oit.uci.edu/~solarese/ee282/iso1_onp_a2_1kb.fastq.gz  

#### 2. Use minimap to overlap reads, use miniasm to construct an assembly and calculate N50 of assembly:  

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

#### 2. Compare your assembly to the contig assembly (not the scaffold assembly!) from Drosophila melanogaster on FlyBase using a dotplot constructed with MUMmer (Hint: use faSplitByN as demonstrated in class)  

module load perl  
module load jje/jjeutils  
module load rstudio  

bioawk -c fastx ' { print length($seq) } ' unitigs.fa   | sort -rn   | awk ' BEGIN { print "Assembly\tLength\nONT_Contig\t0" } { print "ONT_Contig\t" $1 } '   > ONT_Contig_fifo

bioawk -c fastx ' { print length($seq) } ' dmel-all-chromosome-r6.24.fasta   | sort -rn   | awk ' BEGIN { print "Assembly\tLength\nFB_Scaff\t0" } { print "FB_Scaff\t" $1 } ' > r6scaff_fifo  

faSplitByN dmel-all-chromosome-r6-24.fasta dmelcont.fa 10 | bioawk -c fastx ' { print length($seq) } ' dmel-all-chromosome-r6.24.fasta | sort -rn | awk ' BEGIN { print "Assembly\tLength\nFB_Scaff\t0" } { print "FB_Scaff\t" $1 } ' \> r6cont_fifo 

module unload perl  
source /pub/jje/ee282/bin/.qmbashrc  
module load gnuplot  

REF="dmelcont.fa"  
PREFIX="flybase"  
SGE_TASK_ID=1  
QRY=$(ls u*.fa | head -n $SGE_TASK_ID | tail -n 1)  
PREFIX=${PREFIX}_$(basename ${QRY} .fa)  

nucmer -l 100 -c 150 -d 10 -banded -D 5 -prefix ${PREFIX} ${REF} ${QRY}
mummerplot --fat --layout --filter -p ${PREFIX} ${PREFIX}.delta \
  -R ${REF} -Q ${QRY} --postscript  

open postscript file with gv flybasecont_unitigs.ps  

#### 3. Compare your assembly to both the contig assembly and the scaffold assembly from the Drosophila melanogaster on FlyBase using a contiguity plot (Hint: use plotCDF2 as demonstrated in class and see this example):  

_Use fifo filesprepared in previous step_ 

plotCDF2 {ONT_Contig,r6cont,r6scaff}_fifo comparison.png  
display comparison.png

#### 4. Calculate BUSCO scores of both assemblies and compare them

Full Assembly:  
C:98.6%[S:98.1%,D:0.5%],F:0.8%,M:0.6%,n:2799

        2761    Complete BUSCOs (C)
        2747    Complete and single-copy BUSCOs (S)
        14	Complete and duplicated BUSCOs (D)
        21	Fragmented BUSCOs (F)
        17	Missing BUSCOs (M)
        2799    Total BUSCO groups searched

My Assembly:  
C:0.5%[S:0.5%,D:0.0%],F:1.1%,M:98.4%,n:2799

        13	Complete BUSCOs (C)
        13	Complete and single-copy BUSCOs (S)
        0	Complete and duplicated BUSCOs (D)
        32	Fragmented BUSCOs (F)
        2754    Missing BUSCOs (M)
        2799    Total BUSCO groups searched

Running BUSCO Script:  
\#!/bin/bash  
\#  
\#$ -N BuscoUnitigs  
\#$ -q free128,abio128,bio,abio  
\#$ -pe openmp 32  
\#$ -R Y  
module load augustus/3.2.1  
module load blast/2.2.31 hmmer/3.1b2 boost/1.54.0  
source /pub/jje/ee282/bin/.buscorc  

INPUTTYPE="geno"  
MYLIBDIR="/pub/jje/ee282/bin/busco/lineages/"  
MYLIB="diptera_odb9"  
OPTIONS="-l ${MYLIBDIR}${MYLIB}"  
##OPTIONS="${OPTIONS} -sp 4577"  
QRY="unitigs.fa"  
MYEXT=".fa"  

BUSCO.py -c ${NSLOTS} -i ${QRY} -m ${INPUTTYPE} -o $(basename ${QRY} ${MYEXT})_$_${MYLIB}${SPTAG} ${OPTIONS}  

_Replace QRY and MYEXT with necessary information to run for full assembly_


