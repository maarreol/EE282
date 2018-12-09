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
