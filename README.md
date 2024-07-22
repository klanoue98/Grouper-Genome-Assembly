# Grouper Genome Assembly

Records the final workflow for the de novo genome assembly of the red grouper (Epinephelus morio) genome. The purpose of this genome is for future population and aging studies, primarily by the Marine Genomics Lab at TAMUCC. 

The red grouper is expected to have 24 chromosomes, at a genome size of ~1Mbp. 

Additional files that are not publicly available in this repository are available on the Marinegenomics Lab Github

## Data
WD: /work/marinegnomics/klanoue/red_grouper_genome_assembly

Data available for this assembly includes PacBio Hifi reads and HiC data. All original data files can be found on the TAMUCC HPC at /work/marinegenomics/red_grouper_genome. 


#### PacBio Hifi Reads

There are six hifi .bam files available for the red grouper assembly. Each read comes with a .hifi_reads.bam and a .bam.pbi file type. The .bam file contains the needed sequences to be used in genome assembly. The .pbi file is an index file corresponding to its .bam file. 

#### HiC Reads


## 01_Preprocess

Program Fastqc is used to check quality of reads

**Run script pacbioqc.sh**

Run time 03:15:00

### PacBio Read Stats

Total length (bp)
- A20043_11_m64164_230302_051904.fastq.gz: 12529  (32575.4 Mbp)
- A20043_11_m64164_230303_141943.fastq.gz: 13732  (34330 Mbp)
- A20043_11_m64164_230304_232818.fastq.gz: 13642  (34105 Mbp)
- A20043_11_m64164_230306_083357.fastq.gz: 13213  (30389.9 Mbp)
- A20043_11_m64284e_230305_090942.fastq.gz: 12474 (34927.2 Mbp)
- A20043_11_m64292e_230226_035559.fastq.gz: 13951 (32087.3 Mbp)

No. of sequences
- A20043_11_m64164_230302_051904.fastq.gz: 2.6 M
- A20043_11_m64164_230303_141943.fastq.gz: 2.5 M
- A20043_11_m64164_230304_232818.fastq.gz: 2.5 M
- A20043_11_m64164_230306_083357.fastq.gz: 2.3 M
- A20043_11_m64284e_230305_090942.fastq.gz: 2.8 M
- A20043_11_m64292e_230226_035559.fastq.gz: 2.3 M

Total Data
- 193.764453125 Gbp total data 

Estimated genome size
- 828.4 Mbp -> 0.808984375 Gbp

### Bam to FastQ

Program bam2fastx is used to transform PacBio bam files to fastq files. 

**Run script bam2fastx.sh**

Run Time 3:00:00


### Estimate genome characteristics

Run reads through jellyfish and GUI Genomescope to estimate genome size, heterozygosity, and number of unique sequences

**Run script jellyfish.sh**

Run time 01:30:00

Results

Estimated genome haploid length: 828,352,337 bp 
Heterozygosity: 0.486%
Duplicates: 2.25%
Unique sequences: 
K-mer length: 21
Kcoverage: 98.7
Read Error Rate: 0.166571% 

Follow link http://qb.cshl.edu/genomescope/ and input '.histo' file generated by jellyfish into dropbox, press continue, and wait for results Webpage will provide a link so that you can view results anytime.
Results can be reviewed at any time at the following link: http://genomescope.org/analysis.php?code=05wmbofFpYWvrYAIcDcH

### Remove Adapters

PacBio adapters are inserted in the middle of reads during sequencing and should have been removed by the sequencing facility. In the case that some of the adapter may remain, this step will make sure they are removed.

PacBio adapter sequence
5’ – pATCTCTCTCTTTTCCTCCTCCTCCGTTGTTGTTGTTGAGAGAGAT – 3’

**Run script adapter_trim.sh**

Run time: 13:00:00

### Remove mtDNA

Need to make sure that there is not sections of the mitochondrial genome present in the reads.
Download specie's mitochondrial genome from ncbi as a .fasta file, map reads to genome, and remove the reads; only reads not mapped to mitochondrial genome will continue.

**Run script remove_mtdna.sh**

Run time: 04:00:00

Output
final filtered files that will go forward in assembly
  - *.filtered.pt2.fastq.gz
files containing only sequences that successfully mapped to the mitochondrial genome *do not want*
  - mtdna_mapped_*.fastq.gz

Run script blast.sh using both the potential final files and the removed sequences against the red grouper mitochondrial genome (Accession: NC_087999.1)

Script pacbioqc.sh was ran again on the filtered pacbio reads before moving forward to genome assembly. This is not strictly necessary, but interesting to see how the reads change before assembly.

Final reads moving forward to genome assembly

$WORK/preprocess/filter/mtdna_filter/A20043_11_m64284e_230305_090942.fa
$WORKpreprocess/filter/mtdna_filter/A20043_11_m64164_230306_083357.fa
$WORKpreprocess/filter/mtdna_filter/A20043_11_m64292e_230226_035559.fa
$WORKpreprocess/filter/mtdna_filter/A20043_11_m64164_230304_232818.fa
$WORKpreprocess/filter/mtdna_filter/A20043_11_m64164_230303_141943.fa
$WORKpreprocess/filter/mtdna_filter/A20043_11_m64164_230302_051904.fa

## 02_Assemblies

## 2.6.24 wtdbg2

**Run wtbdg2_assembly.sh twice**
 - Once for the assembly and a second time for concensus

Assembly stats

Total kmers = 162470973
 - high frequency kmer depth is set to 3815
 - average kmer depth = 44
 - 22243691 low frequency kmers (<2)
 - 29550 high frequency kmers (>3815)
 - 140197732 kmers indexed, 6193729994 instances (at most)

## 03_Scaffolding

### Arima HiC Alignment

Make sure all required perl scripts are available (can be sourced from MGL GitHub)
- filter_five_end.pl
- two_read_bam_combiner.pl
- get_stats.pl

**Run script hic_align.sh**

Final File for Arima: alignment.bed, move to SALSA

**Run script salsa.sh**

Total Run Time: 04:30:00

Final Data File: scaffolds_FINAL.fasta

## 04_Polishing

**Run script quast.sh**

### Pilon Polisher

A conda environment was created with the required packages: pilonenv

**Run script pilon.sh**

Input genome size according to Pilon: 1032247141
Run Time:16:17:00

Output:
Mean total coverage: 189
Polished genome location: $WORK/04_polish/emo_32124_hifi/pilon/emo_32124_hifi.pilon.fasta

## Final QC

Run quast.sh on the raw assemblies and the scaffolded assemblies

Final Genome Stats

Name: emo_32124_hifi.pilon
- Num. Contigs: 380
- Largest Contig: 41903229
- Total Length: 1027988111
- N50: 34232894
- L50: 14
- Num N's: 53164
