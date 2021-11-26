# Labjournal: antibiotic resistance

## 1) Getting data

Downloaded the data to the dir ./data

### a) Getting the reference sequence of the parental (unevolved, not resistant to antibiotics) E. coli strain
```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/genbank/bacteria/Escherichia_coli/reference/GCA_000005845.2_ASM584v2/GCA_000005845.2_ASM584v2_cds_from_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/genbank/bacteria/Escherichia_coli/reference/GCA_000005845.2_ASM584v2/GCA_000005845.2_ASM584v2_genomic.gff.gz
```
Unzipping:
```bash
gunzip GCA_000005845.2_ASM584v2_cds_from_genomic.fna.gz
gunzip GCA_000005845.2_ASM584v2_genomic.gff.gz
```

### b) Getting raw Illumina sequencing reads from shotgun sequencing of an E. coli strain that is resistant to the antibiotic ampicillin
downloaded from https://figshare.com/articles/dataset/amp_res_2_fastq_zip/10006541, copied to the wd <br>

Unzipping:
```bash
gunzip amp_res_1.fastq.gz
gunzip amp_res_2.fastq.gz
```
Went back to the dir antibiotic-resistance
```bash
cd ../
```
## 2) Manual inspection of the raw sequencing data

### a) Abserving how does the data look like
```bash
head -20 ./data/amp_res_1.fastq
head -20 ./data/amp_res_2.fastq
```

### b) To open the whole file use
```bash
vim file_name.fastq
```

### c) Counting lines in fastq:
```bash
wc -l ./data/amp_res_1.fastq
```
The output: <br>
1823504 ./data/amp_res_1.fastq
```bash
wc -l ./data/amp_res_2.fastq
```
The output:<br>
1823504 ./data/amp_res_2.fastq

## 3) Inspection of the  raw sequencing data with fastqc

installing the program directly from the repository
```bash
sudo apt-get install fastqc
fastqc -h
```
Running fastqc:<br>

fastqc -o /target_dir  /pathtofile1/file1.fastq /pathtofile2/file2.fastq
```bash
fastqc -o . ./data/amp_res_1.fastq ./data/amp_res_2.fastq
```
### a) amp_res_1_fastqc.html:

Basic Statistics: 455876 sequences, 455876 = 1823504/4, so the number of reads it the same<br>
<br>
The red curcles:<br>
- Per base sequence quality (calls of poor quality (red))<br>
The quality of calls on most platforms will degrade as the run progresses, so it is common to see base calls falling into the orange area towards the end of a read.<br>
- Per tile sequence quality (a loss in quality associated with some parts of of the Illumina flowcell)<br>
<br>
There are some unusial points:<br>
- Per base sequence content<br>
- Per sequence GC content<br>
<br>
All other features are ok<br>

### b) amp_res_2_fastqc.html:

Basic Statistics: 455876 sequences, 455876 = 1823504/4, so the number of reads it the same<br>
<br>
The red curcles:<br>
- Per base sequence quality (calls of poor quality (red))<br>
The quality of calls on most platforms will degrade as the run progresses, so it is common to see base calls falling into the orange area towards the end of a read.<br>
<br>
There are some unusial points:<br>
- Per tile sequence quality<br>
- Per base sequence content<br>
- Per sequence GC content<br>
<br>
All other features are ok<br>

## 4) Filtering the reads

### a) Installing the trimming program called Trimmomatic http://www.usadellab.org/cms/?page=trimmomatic
```bash
sudo apt install trimmomatic
TrimmomaticPE
```

### b) the path
```bash
dpkg -L trimmomatic
```
The output:<br>
/usr/share/java/trimmomatic.jar

### c) Running Trimmomatic in paired end mode, with following parameters:

Created a dir for filtered data:
```bash
mkdir ./filtered_data
```
- Cut bases off the start of a read if quality below 20<br>
- Cut bases off the end of a read if quality below 20<br>
- Trim reads using a sliding window approach, with window size 10 and average quality  within the window 20.<br>
- Drop the read if it is below length 20.<br>
```bash
java -jar /usr/share/java/trimmomatic.jar PE -phred33 ./data/amp_res_1.fastq.gz ./data/amp_res_2.fastq.gz amp_res_1_paired.fastq.gz amp_res_1_unpaired.fastq.gz amp_res_2_paired.fastq.gz amp_res_2_unpaired.fastq.gz LEADING:20 TRAILING:20 SLIDINGWINDOW:10:20 MINLEN:20
```
The output:<br>
Input Read Pairs: 455876 Both Surviving: 446259 (97,89%) Forward Only Surviving: 9216 (2,02%) Reverse Only Surviving: 273 (0,06%) Dropped: 128 (0,03%)
```bash
wc -l amp_res_1_paired.fastq
```
The output:<br>
<br>
1785036 amp_res_1_paired.fastq<br>
1785036/4 = 446259<br>
```bash
wc -l amp_res_2_paired.fastq
```
The output:<br>
<br>
1785036 amp_res_2_paired.fastq<br>
1785036/4 = 446259<br>

### d) Inspection of the filtered data with fastqc
```bash
fastqc -o . ./amp_res_1_paired.fastq ./amp_res_2_paired.fastq
```

amp_res_1_paired_fastqc.html: <br>
<br>
Basic Statistics: 446259 sequences, 446259 = 1785036/4, so the number of reads it the same<br>
<br>
The red curcles:<br>
- Per tile sequence quality (a loss in quality associated with some parts of of the Illumina flowcell). We can not fix it because it depend on the tile<br>
<br>
Per base sequence quality is ok now<br>
<br>
The orange curcles:<br>
- Per base sequence content<br>
- Per sequence GC content<br>
- Sequence Length Distribution<br>
<br>
All other features are still ok<br>

amp_res_2_fastqc.html:<br>
<br>
Basic Statistics: 446259 sequences, 446259 = 1785036/4, so the number of reads it the same<br>
<br>
No red curcles anymore<br>
<br>
The orange curcles:<br>
- Per tile sequence quality<br>
- Per base sequence content<br>
- Per sequence GC content<br>
- Sequence Length Distribution<br>

### e) Changing the quality score to 30
```bash
java -jar /usr/share/java/trimmomatic.jar PE -phred33 ./data/amp_res_1.fastq.gz ./data/amp_res_2.fastq.gz amp_res_1_2_paired.fastq.gz amp_res_1_2_unpaired.fastq.gz amp_res_2_2_paired.fastq.gz amp_res_2_2_unpaired.fastq.gz LEADING:30 TRAILING:30 SLIDINGWINDOW:10:30 MINLEN:20
```
The output:<br>
Input Read Pairs: 455876 Both Surviving: 376340 (82,55%) Forward Only Surviving: 33836 (7,42%) Reverse Only Surviving: 25307 (5,55%) Dropped: 20393 (4,47%)<br>
TrimmomaticPE: Completed successfully
```bash
wc -l amp_res_1_paired.fastq
```
1505360 amp_res_1_2_paired.fastq<br>
1505360/4 = 376340
```bash
wc -l amp_res_2_2_paired.fastq
```
1505360 amp_res_1_2_paired.fastq<br>
1505360/4 = 376340
<br>
Fewer reads survive<br>
<br>
Inspection of the filtered data with fastqc<br>
```bash
fastqc -o . ./amp_res_1_2_paired.fastq ./amp_res_2_2_paired.fastq
```
amp_res_1_2_paired_fastqc.html:<br>
<br>
Basic Statistics: 376340 sequences, 376340 = 1505360/4, so the number of reads it the same<br>
<br>
The red curcles:<br>
- Per tile sequence quality (a loss in quality associated with some parts of of the Illumina flowcell). We can not fix it because it depend on the tile<br>
<br>
Per base sequence quality is ok now<br>
<br>
The orange curcles:<br>
- Per base sequence content<br>
- Per sequence GC content<br>
- Sequence Length Distribution<br>
<br>
All other features are still ok<br>
<br>
amp_res_2_2_fastqc.html:<br>
<br>
Basic Statistics: 446259 sequences, 446259 = 1785036/4, so the number of reads it the same<br>
<br>
No red curcles anymore<br>
<br>
The orange curcles:<br>
- Per tile sequence quality<br>
- Per base sequence content<br>
- Sequence Length Distribution<br>
<br>
So we miss more data without critical improvement of the quality<br>

## 5. Aligning sequences to reference

### a) Installing aligner
```bash
brew install bwa
```
### b) Indexing the reference file
```bash
bwa index ./data/GCA_000005845.2_ASM584v2_cds_from_genomic.fna
```
### c) Aligning reads
```bash
bwa mem ./data/GCA_000005845.2_ASM584v2_cds_from_genomic.fna ./filtered_data/qual_score_20/amp_res_1_paired.fastq ./filtered_data/qual_score_20/amp_res_2_paired.fastq > alignment.sam
```
### d) Compressing SAM file

Installing samtools
```bash
brew install samtools
```
Compressing SAM file
```bash
samtools view -S -b alignment.sam > alignment.bam
```
Basic statistics:
```bash
samtools flagstat alignment.bam
```
The output:<br>
<br>
879796 + 0 in total (QC-passed reads + QC-failed reads)<br>
0 + 0 secondary<br>
258 + 0 supplementary<br>
0 + 0 duplicates<br>
878688 + 0 mapped (99.87% : N/A)<br>
879538 + 0 paired in sequencing<br>
439769 + 0 read1<br>
439769 + 0 read2<br>
875578 + 0 properly paired (99.55% : N/A)<br>
877468 + 0 with itself and mate mapped<br>
962 + 0 singletons (0.11% : N/A)<br>
0 + 0 with mate mapped to a different chr<br>
0 + 0 with mate mapped to a different chr (mapQ>=5)<br>
99.87% of reads are mapped. It's good result.<br>

### e) Sorting and indexing BAM file

Sorting bam file by sequence coordinate on reference
```bash
samtools sort alignment.bam -o alignment_sorted.bam
```
Indexing bam file for faster search
```bash
samtools index alignment_sorted.bam
```
### d) Visualizing in IGV
```bash
igv
```
## 6. Variant calling

### a) Creating mpileup
```bash
samtools mpileup -f ./data/genome/GCA_000005845.2_ASM584v2_genomic.fna ./alignment/alignment_sorted.bam > ./varscan/my.mpileup
```
### b) Calling VarScan

I use 0.8 for -min-var-frequency option
```bash
java -jar ../VarScan.v2.4.4.jar mpileup2snp ./varscan/my.mpileup --min-var-freq 0.8 --variants --output-vcf 1 > ./varscan/VarScan_results.vcf
```
Result:<br>
<br>
4641430 bases in pileup file<br>
9 variant positions (6 SNP, 3 indel)<br>
1 were failed by the strand-filter<br>
5 variant positions reported (5 SNP, 0 indel)<br>
