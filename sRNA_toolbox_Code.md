# sRNA toolbox microRNA analysis pipeline

## Software used
| Program | Version | Relevant Links |
| --- | --- | ---|
| Bowtie | 1.3.1 | https://bowtie-bio.sourceforge.net/tutorial.shtml |
| Vienna RNA | 2.6.4 | https://www.tbi.univie.ac.at/RNA/ |
| sRNAtoolboxDB | not specified | https://bioinfo2.ugr.es/srnatoolbox/standalone/ |

## Processing overview with example commands

### 1. Setting up the sRNA database
sRNAbench and sRNAde rely on a local database where most of the library files, genome sequences, Rscripts and Bowtie indexes need to be stored. This database needs both a bowtie index and prepared genome sequence file for the reference annotation. Preprocessing of the genome sequence, with the NCBI parser (https://arn.ugr.es/srnatoolbox/helper/ncbiparser/) may be needed.

#### 1a. Build bowtie index of genome and add to sRNA toolbox index folder
```
bowtie-build genome.fa genome
mv genome.*.ebwt /opt/sRNAtoolboxDB/index/genome.*.ewbt
```
**Parameter Definitions:**
*  genome.fa - reference genome assembly (e.g., GRCh38.p14) from NCBI
*  genome - prefix that will be used for the bowtie index files  

**Input Data:**
*  genome.fa - reference genome assembly  

**Output Data:** 
*  genome.*.ebwt - bowtie index files (1, 2, 3, 4, rev.1, rev.2)

#### 1b. Create prepared genome sequence and add to seqOBJ folder
```
java -jar makeSeqObj.jar genome.fa
mv genome.zip /opt/sRNAtoolboxDB/seqOBJ/genome.zip
```
**Parameter Definitions:**
*  genome.fa - reference genome assembly (e.g., GRCh38.p14) from NCBI

**Input Data:**
*  genome.fa - reference genome assembly

**Output Data:** 
*  genome.zip - prepared genome sequences for sRNAbench

### 2. Preprocessing and adapter removal
May need to add the bowtie-1.3.1 folder to PATH for sRNA toolbox (i.e., export PATH=$PATH:/path/to/bowtie-1.1.1)
```
java -jar /opt/sRNAtoolboxDB/exec/sRNAbench.jar input=/path/to/*.fq output=/opt/sRNAtoolboxDB/out/pre/ adapterMinLength=6 adapter=TCGTATGCCG 
```
**Parameter Definitions:**
*  *.fq - input reads
*  adapterMinLength - the minimum length of the adapter that needs to be detected
*  adapter=TCGTATGCCG - the adapter sequence. If this parameter is NOT given on the command line, then the input is assumed to be adapter trimmed already

**Input Data:**
*  *.fq (raw reads)

**Output Data:** (within out/pre folder) 
*  logFile.txt - Different analysis steps are logged, but additionally possible warnings and errors are written to this file.  
*  parameters.txt - list of parameters used in the sRNAbench.jar step
*  results.txt - The results of the different steps, i.e. preprocessing
*  reads_orig.fa - Reads after the preprocessing, i.e. after adapter trimming, length and quality filtering (default min PhredScore 20) and collapsing
*  reads.fa - Reads that have not been mapped to the miRBase reference, this file is automatically created even though this prepocessing step did not include mapping reads, so it is technically the same as reads_orig.fa
*  short_reads.txt - the reads filtered out due to minReadLength parameter (the default is 15nt)
* stat/readLengthAnalysis.txt -  distribution of the reads that are used for the analysis
* stat/readLengthFull.txt -  length distribution without setting any thresholds like minimum length or minimum read count
 
### 3. microRNA profiling with genome mapping mode
```
java -jar /opt/sRNAtoolboxDB/exec/sRNAbench.jar input=/opt/sRNAtoolboxDB/out/pre/reads_orig.fa output=/opt/sRNAtoolboxDB/out/miR microRNA=hsa species=genome
```
**Parameter Definitions:**
*  reads_orig.fa - Reads after the preprocessing
*  microRNA - short species name used in miRBase (e.g., hsa, mmu), more than one species can be selected separating them by ‘:’
*  species - the name of the bowtie index in 'index' folder

**Input Data:**
*  reads_orig.fa - Reads after the preprocessing

**Output Data:** (within out/miR folder) 
*  logFile.txt - Different analysis steps are logged, but additionally possible warnings and errors are written to this file.  
*  parameters.txt - list of parameters used in the sRNAbench.jar step
*  results.txt - The results of the different steps, i.e. preprocessing
*  genomeDistribution.txt - the number of reads mapped to the different species specified in microRNA parameter
*  genomeDistribution folder -  contains the reads assigned to the different assemblies in fasta format and the corresponding read length distribution.
*  genomeMappedReads.fa - the reads mapped to any of the genome assemblies (only non-redundant reads are included)
*  genomeMappedReads.readLen - the read length distribution of genome mapped reads

### 4. Prediction of novel microRNAs
```
java -jar /opt/sRNAtoolboxDB/exec/sRNAbench.jar input=/opt/sRNAtoolboxDB/out/pre/reads_orig.fa output=/opt/sRNAtoolboxDB/out/prediction microRNA=hsa species=genome predict=true minReadLength=19 maxReadLength=25
```
**Parameter Definitions:**
*  reads_orig.fa - Reads after the preprocessing
*  microRNA - short species name used in miRBase (e.g., hsa, mmu), more than one species can be selected separating them by ‘:’
*  species - the name of the bowtie index in 'index' folder
*  predict=true - calls the prediction of novel microRNAs function, default assumes animals for plants kingdom=plants should be included
*  minReadLength - minimum read length in nt
*  maxReadLength - maximum read length in nt

**Input Data:**
*  reads_orig.fa - Reads after the preprocessing

**Output Data:** (within out/prediction folder) 
*  logFile.txt - Different analysis steps are logged, but additionally possible warnings and errors are written to this file.  
*  parameters.txt - list of parameters used in the sRNAbench.jar step
*  results.txt - The results of the different steps, i.e. preprocessing
*  novel.txt - Summary of novel microRNAs
*  novel_mature.fa and novel_hairpin.fa - mature and pre-microRNA sequences of novel microRNAs
*  folder novel - contains the alignments to the novel pre-microRNA sequences 
  
### 5. Detecting and Classifying IsomiRs and IsoRNA:
```
java -jar /opt/sRNAtoolboxDB/exec/sRNAbench.jar input=/opt/sRNAtoolboxDB/out/pre/reads_orig.fa output=/opt/sRNAtoolboxDB/out/libs microRNA=hsa isoMiR=true
```
**Parameter Definitions:**
*  reads_orig.fa - Reads after the preprocessing
*  microRNA - short species name used in miRBase (e.g., hsa, mmu), more than one species can be selected separating them by ‘:’
*  isoMiR=true - detects IsomiRs

**Input Data:**
*  reads_orig.fa - Reads after the preprocessing

**Output Data:** (within out/prediction folder) 
*  logFile.txt - Different analysis steps are logged, but additionally possible warnings and errors are written to this file.  
*  parameters.txt - list of parameters used in the sRNAbench.jar step
*  results.txt - The results of the different steps, i.e. preprocessing
*  mature.iso: isomiR information for each mature microRNA.
*  microRNAannotation: isomiR annotation at a read level.
*  isomiR_summary.txt (in stat folder): The number of reads found for all isomiR classes for each of the mature microRNAs.
*  isomiR_NTA.txt and isomiR_otherVariants.txt (in 'stat' folder): isomiR summary of the sample.
*  isomiR_lenNTA.txt (in ‘stat’ folder): summary of non-templated additions as a function of 'addition length'
 
### 6. Differential expression
```
java -jar /opt/sRNAtoolboxDB/exec/sRNAde.jar input=/opt/sRNAtoolboxDB/out/ output=/path/to/DE/out/ grpString=<the names of the input samples> diffExpr=true
```
**Parameter Definitions:**
*  input - the path to the sRNAbench output folders of the individual sRNAbench runs
*  output - the path of the output folder
*  grpString - the group string must contain the names of the different sRNAbench output folders in the following way: f1_1:f2_1#f1_2:f2_2 being f1_1 the first folder (sample) of the first group (controls in a case/control study) f2_1 the the second folder of the first group, f1_2 the first folder of the second group (cases) etc
*  diffExpr=true - perform differential expression analyses

**Input Data:**
*  input - the path to the sRNAbench output folders of the individual sRNAbench runs

**Output Data:** 
*  *.xlsx - The output generated by edgeR, DEseq and NOI-seq
*  *.mat - the adjusted read count expression matrix which is used as input for edgeR, DEseq and NOI-seq
*  *totalRPM.mat - The normalized read count matrix (normalized with the total number of reads)
*  *libraryRPM.mat: The normalized read count matrix (normalized with the number of reads mapped to the corresponding library – microRNAs in this case)
*  *TMM_normalized.tsv: the TMM normalized expression matrix (generated by edgeR)
*  heatmap.png -  the cluster analysis and heatmap taking the top 50 microRNAs
*  heatmap_median_normalized.png: the cluster analysis and heatmap taking the top 50 microRNAs applying median normalization

### 7. Concnesus miRNA target predictions
This program allows to calculate consensus miRNA target predictions, both in animals and plants. Must be downloaded directly from the github repository (https://github.com/bioinfoUGR/sRNAtoolbox/tree/master/exec)
```
java -jar miRNAconsTargets.jar
java -jar miRNAconsTargets_plants.py
```
