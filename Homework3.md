## Summarize a genome assembly  

**Download required files:**  
wget ftp://ftp.flybase.net/genomes/Drosophila_melanogaster/dmel_r6.24_FB2018_05/fasta/dmel-all-chromosome-r6.24.fasta.gz  
wget ftp://ftp.flybase.net/genomes/Drosophila_melanogaster/dmel_r6.24_FB2018_05/fasta/md5sum.txt

**Checksum:**  
md5sum dmel-all-chromosome-r6.24.fasta.gz 
  
_Compare to md5sum.txt file to see if there is an exact match. If not then delete and redownload file._  
Unzip file: gunzip dmel-all-chromosome-r6.24.fasta.gz

**Calculate Total Number of Nucleotides:**  
grep -io [agtnc] dmel-all-chromosome-r6.24.fasta | wc -l

**Calculate the total number of Ns:**  
grep -io [n] dmel-all-chromosome-r6.n4.fasta | wc -l  

**Calculate the number of sequences:**  
grep "^>" dmel-all-chromosome-r6.24.fasta | wc -l

## Summarize an annotation file
**Download required files:**  
wget ftp://ftp.flybase.net/genomes/Drosophila_melanogaster/dmel_r6.24_FB2018_05/gtf/dmel-all-r6.24.gtf.gz  
wget ftp://ftp.flybase.net/genomes/Drosophila_melanogaster/dmel_r6.24_FB2018_05/gtf/md5sum.txt  

_Make sure to remove old md5sum.txt file so as to not compare old checksum to new one_  

**Checksum:**  
md5sum dmel-all-r6.24.gtf.gz  

_Compare to md5sum.txt file to see if there is an exact match. If not then delete and redownload file._  
Unzip file: gunzip dmel-all-chromosome-r6.24.fasta.gz

_**Print a summary report with the following information:**_  

**Total number of features of each type, sorted from the most common to the least common**  
grep -v "^#" dmel-all-r6.24.gtf | cut -f3 | sort | uniq -c | sort -rn  

**Total number of genes per chromosome arm (X, Y, 2L, 2R, 3L, 3R, 4)**  
grep -v "^[#]" dmel-all-r6.24.gtf | cut -f1 | sort | uniq -c | sort -rn | head -7

### Comments

You could do the comparison with the checksum automatically by using the following:

```
md5sum -c md5sum.txt
```
It automatically goes through all files in the md5sum.txt file and compares the computed checksum of the local files to what's in the file. What you wrote works, but this is better.

As for counting genes on chromosomes, you forgot to filter for only gene features.

```
grep -v "^[#]" dmel-all-r6.24.gtf | awk ' $3 == "gene" ' | cut -f1 | sort | uniq -c | sort -rn | head -7
```
