# Introduction to Metagenomics

---------

This tutorial is adapted from ["Metagenomics tutorial part 1"](https://usda-ars-gbru.github.io/Microbiome-workshop/tutorials/metagenomics/),  by  Adam Rivers.

---------

MetaGenomics is the study of genetics from the material directly obtained from environmental samples without the use of cultivation-based methods. Shotgun metagenomics is an approach based on randomly breaking down a long sequence of DNA fragment, use sequencing methods ( such as sanger or NGS) to sequence each fragment (contigs), and use algoritms to overlap the fragments to get the complete range of sequence.  

# Overview
Shotgun metagenomics data can be analyzed using several different approaches. Each approach is best suited for a particular group of questions. The methodological approaches can be broken down into three broad areas: read-based approaches, assembly-based approaches and detection-based approaches. This tutorial takes an assembly-based approach. The key points of the approaches are listed in this table:

Method | Read-based | Assembly-based | Detection-based
-------|------------|----------------|----------------
Description | Read-based metagenomics analyzes unassembled reads. It was one of the first methods to be used. It is still valuable for quantitative analysis, especially if relevant references are available. | Assembly-based workflows attempt to assemble the reads from one or more samples, "bin" contigs from these samples into genomes then analyze the genes and contigs. | Detection-based workflows attempt to identify with very high-precision but lower sensitivity (recall) the presence of organisms of interest, often pathogens.
Typical workflow | 1. Read QC.<br>2. Read merging (Bbmerge).<br>3. Mapping to NR for taxonomy data (Blast, Diamond or Last). Kmer-based approaches are also used (Kraken, Clarke).<br> 4. Mapping to functional databases (Kegg, Pfam). Cross-mapping can also be used (Mocat2).<br> 5. Summaries of taxonomic and functional distributions can be analyzed. If read IDs were retained function within taxa analyses can be run.<br> | 1. Read QC.<br>2. Read merging.<br>3. Read assembly. (Megahit, Spades-meta). Generally individual samples are assembled.  Memory use is an issue. Read-error correction and normalization can help.<br> 4. Mapping of reads reads from each sample are mapped back to the contigs from all samples for use in quantification and binning (bowtie2, bbmap).<br> 5. Genome binning. Contig composition and mapping data are used to bin contigs into "genome bins" (Metabat, MaxBin, Concoct).<br> 6.  <em>De-novo</em> gene annotation is done (Prodigal, MetaGeneMArk). Gene annotation is performed as in read-centric approaches, but computational burden is lower since annotated data is ~100x smaller.<br> 7. Analysis of genome bins, functional gene phylogenetics, comparative genomics. | 1. Read QC<br>2. Read merging (if paired) <br> 3. Alignment or kmer based matching against curated dataset.<br> 4. Heuristics for determining the correct level to classify at and the  classification (Megan-LCA, Clarke, Surpi, Centrifuge, proprietary methods).
Typical questions|&bull; What is the bulk taxonomic/functional composition of these samples?<br>&bull; What new kinds of a particular functional gene family can I find?<br>&bull; How do my sites or treatments differ in taxonomic/functional composition?<br> |&bull; What are the functional and metabolic capabilities of specific microbes in my sample?<br>&bull; What is the phylogeny of gene families in my samples?<br>&bull; Do the organisms that inhabit my samples differ?<br>&bull; Are there variants within taxa in my population? | &bull; Are known organisms of interest present in my sample?<br> &bull; Are known functional genes e.g. beta-lactamases, present in my sample?<br>
Examples | [MG-Rast](http://metagenomics.anl.gov/) | [IMG from JGI](https://img.jgi.doe.gov/cgi-bin/mer/main.cgi) | [Taxonomer](http://taxonomer.iobio.io/), [Surpi](http://chiulab.ucsf.edu/surpi/), [One Codex](https://www.onecodex.com/), [CosmosID](http://www.cosmosid.com/)

### Data sets in this tutorial

The data set for this tutorial is from a mock viral community from [Adam Rivers' tutorial page](https://usda-ars-gbru.github.io/Microbiome-workshop/tutorials/metagenomics/). The data file can be downloaded from [here](https://usda-ars-gbru.github.io/Microbiome-workshop/assets/metagenome/10142.1.149555.ATGTC.subset_500k.fastq.gz).

Many of the initial processing steps in metagenomics are quite computationally intensive. For this reason, it is recommended that you use computer clusters through out this tutorial.

### Connecting to computer cluster

 From Terminal or Putty (for Windows users) create a secure shell connection to your computer cluster.

You can connect to any remote machine that you have access to:

```bash
ssh -o TCPkeepAlive=<yes or No> -o ServerAliveInterval=<timeout interval> -o ServerAliveCountMAx=<number of messages> <username><computer name or the IP address>
```

options:
* `TCPkeepAlive`: Ensures that certain firewalls don't drop idle connections

* `ServerAliveInterval`: Sets a timeout interval in seconds after which if no data has been received from the server, ssh will send a message through the encrypted channel to request a response from the server. The default is 0, indicating that these messages will not be sent to the server.

* `ServerAliveCountMAx`:  Sets the number of server alive messages which may be sent without ssh receiving any messages back from the server. If this threshold is reached while server alive messages are being sent, ssh will disconnect from the server, terminating the session.The default value is 3. If, for example, ServerAliveInterval  is set to 15 and ServerAliveCountMax is left at the default, if the server becomes unresponsive, ssh will disconnect after approximately 45 seconds.


Once you are logged in, you can request access to an interactive node. In a real analysis you would create a script that runs all the commands in sequence and submit the script through a program (such as [sbatch](https://slurm.schedmd.com/sbatch.html)) that submits a batch script to a computer's job scheduling software such as [Slurm](https://slurm.schedmd.com/)).

To request access to an interactive node on [slurm workload manager](https://slurm.schedmd.com/):
```bash
salloc -p <queue(partition) name> -N <number of nodes> -n <number of tasks> -t <request  time in hour>
```


*  To see available queues on [slurm workload manager](https://slurm.schedmd.com/) use the command "sinfo"
* The default value for -n is one task per node, but  the --cpus-per-task option will change this default.
* Request time as hh:mm:ss , example : 04:00:00 (four hours)



### Required modules

* [BBDuk](https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbduk-guide/) : for trimming
* [Megahit](https://github.com/voutcn/megahit): for assembly
* [samTools](http://samtools.sourceforge.net/) : for sequence alignment
* [Quast](http://bioinf.spbau.ru/quast) : for evaluating genome assemblies

 To check for currently loaded modules:
 ```
module list
 ```
 To get a list of available modules:
 ```
module avail
 ```
 To load a module:
 ```
module load <module name>
 ```

## Part 1: Assembly and mapping

Create a directory for the tutorial.

```bash
# In your homespace or other desired location, make a
# directory and move into it
mkdir metagenome1
cd metagenome1
# make a data directory
mkdir data
cd data
```

Lets take a peek at the data.
```bash
zcat 10142.1.149555.ATGTC.subset_500k.fastq.gz | head
```
The output should look like this:
```
@MISEQ05:522:000000000-ALD3F:1:2114:16371:19347 1:N:0:ATGTC
GGATATTTTGCTTTAAAATTTTCTTTTACCGCTTTTGGCACAGCATCATCCTGATATTGACAACTCATATACGTTACCGAAACAAACAAAGTCAAAAATAATAAACAGCTTCTCATGGCTTTCTTTTTTAAATTCTAAGCTGCGAAGAAAC
+
CCCDCFFFFFFFGGGGGGGGGGHHHHHHHHGGGGGHHGHHHHHHHGHHHHHHHHHHHIHHHHHHHHHHHHHHHHHHGHGGGGGHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHGHHHHHHHHHHHHHHGGGGGHHH
@MISEQ05:522:000000000-ALD3F:1:2114:16371:19347 2:N:0:ATGTC
GAGAGAAGAGATATAGGACTAAAATGTATTTTTTTTACTGCTAGTGCTTTTGAGCTAGTAATCGTTTTTATAAAGTTAACTATTGAGGGTGCTAATCGGTTTCTTCGCAGCTTAGAATTTAAAAAAGAAAGCCATGAGAAGCTGTTTATTA
+
CCCCCDFFFFFFGGGGGGGGGGHHHHHHHHHHHGGGHHHHHHHHHHHHHHHHGGHHHHHHHHHGHHGGGHHHHHFHHHGHHHHHHHHHGGFGHHHGHGGGGGHHHHHGGGGHHHHHHHHHHHHHHGGGHHHHGHHHHHHHHHHHHHHHHHH
@MISEQ05:522:000000000-ALD3F:1:2114:17821:9269 1:N:0:ATGTC
AATTTTACCTTTTGATGCAAATTTAAACTTAACGAAAGCGTCTGTAGTATCTTTTAAACTAAGACCCATTTCATCAACAATTCCATTTATGAACATCATATCCTTTGCAGCAGCTTCCGAACTACCACTTGCAGCAAGCATAGAAGCCTCC
```

We can see that this data is interleaved, paired-end based on the 1 and 2 after the initial identifier. From the identifier and the length of the reads we can see that the data was sequenced in 2X150  mode on a Illumina Miseq instrument.

Raw sequencing data needs to be processed to remove artifacts. The first step in this process is to remove contaminant sequences that are present in the sequencing process such as PhiX which is sometimes added as an internal control for sequencing runs:

### Kmer filtering


```bash
# Filter out contaminant reads placing them in their own file

bbduk.sh in=10142.1.149555.ATGTC.subset_500k.fastq.gz\
out=10142.1.149555.ATGTC.subset_500k.unmatched.fq.gz \
outm=10142.1.149555.ATGTC..subset_500k.matched.fq.gz \
k=31 \
hdist=1 \
ftm=5 \
ref=<sequencing_artifacts.fa.gz file including path> \
stats=contam_stats.txt
```
This will remove all reads that have a 31-mer match to the artifacts reference reads, allowing 1 mismatch.

* in=  : input file
* out= : file containing filtered Reads
* outm= : file containing reads that match the reference kmer
* k= : Kmer number
* hdist= : hamming distance, the number of allowed mismatches
* ftm= modulo trim number ( usually 5), the right end is trimed so that the read's length is equal to molulo this number. The reason is that with Illumina sequencing runs are usually a multiple of 5 in length but sometimes they are generated with an extra base. This last base is very inaccurate and has badly calibrated quality as well, so it’s best to trim it before doing anything else.
* ref= :  A file with all sequencing artifacts included in software source. can be found in  ```resources/``` directory where the bbToolkit program is installed. You can contact your cluster computer help desk for locating the file.  

Lets look at the output:

```
BBDuk version 37.36
Initial:
Memory: max=53667m, free=52547m, used=1120m

Added 8085859 kmers; time:      1.298 seconds.
Memory: max=53667m, free=50027m, used=3640m

Input is being processed as paired
Started output streams: 0.087 seconds.
Processing time:                3.996 seconds.

Input:                          500000 reads            75500000 bases.
Contaminants:                   1044 reads (0.21%)      156600 bases (0.21%)
FTrimmed:                       500000 reads (100.00%)  500000 bases (0.66%)
Total Removed:                  1044 reads (0.21%)      656600 bases (0.87%)
Result:                         498956 reads (99.79%)   74843400 bases (99.13%)

Time:                           5.409 seconds.
Reads Processed:        500k    92.44k reads/sec
Bases Processed:      75500k    13.96m bases/sec

```
0.21% of our reads contained contaminants or were too short after low quality reads were removed, suggesting that our insert size selection went well.

If we look at the `contam_stats.txt` file we see that most of the contaminants were known Illumina linkers. Because so many of these have been erroneously deposited in the NT database, Blasting the contaminant sequences will turn up some very odd things. The first sequence in our contaminants file is supposedly a Carp.
```
#File   10142.1.149555.ATGTC.subset_500k.fastq.gz
#Total  500000
#Matched        884     0.17680%
#Name   Reads   ReadsPct
contam_27       477     0.09540%
contam_11       240     0.04800%
contam_12       75      0.01500%
contam_10       59      0.01180%
contam_9        16      0.00320%
contam_257      8       0.00160%
contam_175      2       0.00040%
contam_256      2       0.00040%
contam_102      1       0.00020%
contam_15       1       0.00020%
contam_178      1       0.00020%
contam_181      1       0.00020%
contam_8        1       0.00020%

```

With shotgun paired end sequencing the size of the DNA sequenced is controlled by physically shearing the DNA and selecting DNA of a desired length with SPRI or Ampure beads.  This is an imperfect process and sometimes short pieces of DNA are sequenced.  If the DNA is shorter than the sequencing length (150pb in this case) The other adapter will be sequenced too.

### Adapter trimming

We can trim off this adapter and low quality ends using bbduk.

```bash
# Trim the adapters using the reference file adaptors.fa (provided by bbduk)

bbduk.sh in=data/10142.1.149555.ATGTC.subset_500k.unmatched.fq.gz \
out=data/10142.1.149555.ATGTC.subset_500k.trimmed.fastq.gz \
ktrim=r \
k=23 \
mink=11 \
hdist=1 \
tbo=t \
qtrim=r \
trimq=20 \
ref=<adapters.fa with full path>
```

* ktrim=r : right triming ( 3' adaptor)
* k=23 : Kmer number
* mink=11 : Adopters' length is usually shorter than the Kmer. this option additioanlly looks for shorter kmers between mink number and kmer number at the end of the read.
* hdist=1 : hamming distance
* tbo=t :  trim adopters based on where paired reads overlap

Looking at the output
```
BBDuk version 37.36
maskMiddle was disabled because useShortKmers=true
Initial:
Memory: max=53667m, free=52547m, used=1120m

Added 216529 kmers; time:       0.152 seconds.
Memory: max=53667m, free=50027m, used=3640m

Input is being processed as paired
Started output streams: 0.073 seconds.
Processing time:                4.365 seconds.

Input:                          498956 reads            74843400 bases.
QTrimmed:                       40363 reads (8.09%)     1140629 bases (1.52%)
KTrimmed:                       3356 reads (0.67%)      273370 bases (0.37%)
Trimmed by overlap:             1954 reads (0.39%)      9950 bases (0.01%)
Total Removed:                  2118 reads (0.42%)      1423949 bases (1.90%)
Result:                         496838 reads (99.58%)   73419451 bases (98.10%)

Time:                           4.613 seconds.
Reads Processed:        498k    108.17k reads/sec
Bases Processed:      74843k    16.23m bases/sec
[msayadi@condo153 toturial]$ salloc: Job 353318 has exceeded its time limit and its allocation has been revoked.
srun: Job step aborted: Waiting up to 32 seconds for job step to finish.
srun: error: condo153: task 0: Killed

```
* 40363 reads had their right ends quality trimmed
* 3356 reads had adaptors trimmed because the matched they matched adaptors in the database
* 1954 reads had a few base pairs trimmed because a small portion of an adaptor was detected when the read pairs were provisionally merged.
* 2118 reads were removed because after trimming they were too short.

## Merging Reads

Overlapping reads are sometimes occurred in pair-end reads if the fragment length(F) is shorter than double the size of the reads(L).  Many software tools get confused with overlapping pairs. By merging the overlapping pairs and turning them into longer single-end reads many tools will produce better results.  

In this toturial we use [BBMerge](https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbmerge-guide/) from BBTools toolkit.

```bash
# Merge the reads together
bbmerge.sh in=10142.1.149555.ATGTC.subset_500k.trimmed.fastq.gz \
out=10142.1.149555.ATGTC.subset_500k.merged.fastq.gz \
outu=10142.1.149555.ATGTC.subset_500k.unmerged.fastq.gz \
ihist=insert_size.txt
```
* in= input file
* out= the output file for overlapped Reads
* outu= if no best overlap is found the pair will go to outu
* ihist= the insert size histogram file generated from the reads

Here's the output:
```
BBMerge version 37.36
Writing mergable reads merged.
Started output threads.
Total time: 3.347 seconds.

Pairs:                  248419
Joined:                 132258          53.240%
Ambiguous:              116124          46.745%
No Solution:            37              0.015%
Too Short:              0               0.000%

Avg Insert:             234.4
Standard Deviation:     34.9
Mode:                   244

Insert range:           120 - 291
90th percentile:        278
75th percentile:        262
50th percentile:        238
25th percentile:        211
10th percentile:        186

```
From the summary we can see that our average insert size was 234bp and 53% of
reads could be joined. From the insert-size.txt histogram file we can plot the
data and see the shape of the distribution.

![Insert Size Histogram Plot](assets/metagenome/insertsize.svg)


## Metagenome assembly

Most of our cleanup work is now done. We can assemble the metagenome with the Succinct DeBruijn graph assembler [Megahit](https://doi.org/10.1093/bioinformatics/btv033).

```bash
time megahit  -m 1200000000000 \
-t 38 \
--read data/10142.1.149555.ATGTC.subset_500k.merged.fastq.gz \
--12 data/10142.1.149555.ATGTC.subset_500k.unmerged.fastq.gz \
--k-list 21,41,61,81,99 \
--no-mercy \
--min-count 2 \
--out-dir data/megahit/
```
Time to run: 3 minutes

# Looking at the assembly

Megahit tells us that the longest contig was 129514 bp and the N50 was 6252 bp. We can use Quast to look at the assembly more.

```bash
quast.py -o data/quast -t 4 -f --meta data/megahit/final.contigs.fa
```
Time to run: 10 seconds

All statistics are based on contigs of size >= 500 bp, unless otherwise noted (e.g., "# contigs (>= 0 bp)" and "Total length (>= 0 bp)" include all contigs).



Assembly |final.contigs
---------|-----------------------------
\# contigs (>= 0 bp)      |1360
\# contigs (>= 1000 bp)   |1011
Total length (>= 0 bp)   |4993279
Total length (>= 1000 bp)|4784908
\# contigs                |1231
Largest contig           |129514
Total length             |4945499
GC (%)                   |34.94
N50                      |6252
N75                      |3311
L50                      |200
L75                      |470
\# N's per 100 kbp         |0.00

![Cumulative contig graph](assets/metagenome/cumulative_plot.svg)

Although its not generally required, we can also visualize our assemblies by generating a fastg file

```bash
megahit_toolkit contig2fastg 99 \
data/megahit/intermediate_contigs/k99.contigs.fa > data/k99.fastg
```
Time to run: 3 seconds

Download the file to  your local computer by opening a new Terminal window and copying the file

```bash
scp user.name@login.ecinet.science:~/metagenome1/data/k99.fastg .
```

Now open Bandage and select **File > Open Graph** and load ```k99.fastg```

Hit the **Draw Graph** button. Now all the assembled contigs are visible. Some are circular, some are linear and some have bubbles in the assembly.

![Main Bandage Screen](assets/metagenome/Bandage1.png)

IF you have a Ssand-alone copy of [Blast+](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download) installed on your computer you can label the contigs with with the reference genomes.

Download the reference fasta file here:
Ref_database.fna : [Download](https://usda-ars-gbru.github.ioassets/metagenome/Ref_database.fna)

Select **Build blast database**
Select **Load from Fasta File** and load ```Ref_database.fna```

Select **Run Blast Search** then **Close**
![Blast options screen](assets/metagenome/Bandage2.png)

The contigs will now be taxonomically annotated. Take some time to explore the structure of the contigs. Which ones look correct? What kinds or errors are present in the assembly?

What are the contigs that assembled but were not identified by Blasting to our reference database? The fourth largest node ,Node 987 is actually genomic DNA from the bacteria *Cellulophaga baltica* the host of the 6 of the phages in our metagenome. Our protocol can detect and assemble this host DNA even though its present at 10-100 times lower abundance than the phages in our sample.

If we go back to the Ceres command line we can continue with the next common step in read based assembly, mapping. Two common types of mapping can occur. You can map reads back to a known reference. This is great if you have closely related organisms of interest and you are trying to understand population variance or quantify known organisms.  A number of programs can be used  for mapping. Bowtie2 is the most common but in this case we will use bbmap.

Map reads back to known reference
```bash
bbmap.sh \
  ref=/project/microbiome_workshop/metagenome/viral/data/Ref_database.fna \
  in=data/10142.1.149555.ATGTC.subset_500k.trimmed.fastq.gz \
  out=data/10142.1.149555.ATGTC.map_to_genomes.sam \
  nodisk \
  usejni=t \
  pigz=t \
  threads=40 \
  bamscript=bamscript1.sh
```
Time to run: 1 minute

When the results are mapped to a reference we are able to infer the  Error rate for insertions, substitutions and deletions in our sequences. Almost all of these errors come from sequencing itself, not assembly. Note too, how much higher the error rate is for Read 2 than Read 1.

We can also see the insert size for those reads that did not merge is 300.
```

   ------------------   Results   ------------------

Genome:                	1
Key Length:            	13
Max Indel:             	16000
Minimum Score Ratio:  	0.56
Mapping Mode:         	normal
Reads Used:           	496904	(73428502 bases)

Mapping:          	303.347 seconds.
Reads/sec:       	1638.07
kBases/sec:      	242.06


Pairing data:   	pct reads	num reads 	pct bases	   num bases

mated pairs:     	 47.1367% 	   117112 	 47.5180% 	    34891788
bad pairs:       	  0.1139% 	      283 	  0.1111% 	       81554
insert size avg: 	  300.37


Read 1 data:      	pct reads	num reads 	pct bases	   num bases

mapped:          	 47.7621% 	   118666 	 47.7346% 	    17650903
unambiguous:     	 47.7497% 	   118635 	 47.7229% 	    17646580
ambiguous:       	  0.0125% 	       31 	  0.0117% 	        4323
low-Q discards:  	  0.0000% 	        0 	  0.0000% 	           0

perfect best site:	 43.3645% 	   107740 	 43.4335% 	    16060475
semiperfect site:	 43.4748% 	   108014 	 43.5444% 	    16101468
rescued:         	  3.1821% 	     7906

Match Rate:      	      NA 	       NA 	 99.4266% 	    17612450
Error Rate:      	  8.9689% 	    10643 	  0.4979% 	       88197
Sub Rate:        	  8.3891% 	     9955 	  0.1328% 	       23519
Del Rate:        	  0.2284% 	      271 	  0.3563% 	       63115
Ins Rate:        	  0.6143% 	      729 	  0.0088% 	        1563
N Rate:          	  0.2958% 	      351 	  0.0755% 	       13371


Read 2 data:      	pct reads	num reads 	pct bases	   num bases

mapped:          	 47.3202% 	   117568 	 47.4124% 	    17282474
unambiguous:     	 47.3057% 	   117532 	 47.4002% 	    17278009
ambiguous:       	  0.0145% 	       36 	  0.0122% 	        4465
low-Q discards:  	  0.0000% 	        0 	  0.0000% 	           0

perfect best site:	  1.3971% 	     3471 	  1.3724% 	      500267
semiperfect site:	  1.4486% 	     3599 	  1.4246% 	      519288
rescued:         	  0.4162% 	     1034

Match Rate:      	      NA 	       NA 	 91.5571% 	    16625718
Error Rate:      	 96.8691% 	   113888 	  8.3635% 	     1518716
Sub Rate:        	 95.1773% 	   111899 	  3.3643% 	      610918
Del Rate:        	  7.0826% 	     8327 	  4.8262% 	      876386
Ins Rate:        	  7.7478% 	     9109 	  0.1730% 	       31412
N Rate:          	  0.2705% 	      318 	  0.0794% 	       14426
```

The mapper writes text-based sam mapping files. It's usually helpful to convert
these files to a binary format called bam which is faster to access and smaller. Fortunately, bbmap will automatically make a little script to do that for us using the program Samtools. To run it enter this command.

```bash
./bamscript1.sh
```

It is also possible to map reads back to the contigs we just generated. Reads are mapped back to contigs for several reasons. Once genes are called gene counts can be derived from mapping data. Contig mapping data is also used in genome binning.

Map reads back to known reference
```bash
bbmap.sh \
  ref=data/megahit/final.contigs.fa \
  in=data/10142.1.149555.ATGTC.subset_500k.trimmed.fastq.gz \
  out=data/10142.1.149555.ATGTC.map_to_contigs.sam \
  nodisk \
  usejni=t \
  pigz=t \
  threads=40 \
  bamscript=bamscript2.sh
```
Time to run: 1 minute

With the contigs the match-rate was much higher. 95% of our reads map back to the assembly which is a very high number. in a well assembled complex metagenome we might expect 50-70% of reads to map.

```
Pairing data:   	pct reads	num reads 	pct bases	   num bases

mated pairs:     	 97.0034% 	   241007 	 97.8497% 	    71849602
bad pairs:       	  1.1938% 	     2966 	  1.1811% 	      867288
insert size avg: 	  298.83


Read 1 data:      	pct reads	num reads 	pct bases	   num bases

mapped:          	 99.1592% 	   246363 	 99.1688% 	    36669800
unambiguous:     	 99.1447% 	   246327 	 99.1548% 	    36664611
ambiguous:       	  0.0145% 	       36 	  0.0140% 	        5189
low-Q discards:  	  0.0000% 	        0 	  0.0000% 	           0

perfect best site:	 90.2730% 	   224285 	 90.4738% 	    33454627
semiperfect site:	 90.7552% 	   225483 	 90.9576% 	    33633528
rescued:         	  6.4733% 	    16083

Match Rate:      	      NA 	       NA 	 99.6846% 	    36578005
Error Rate:      	  8.4838% 	    20901 	  0.2057% 	       75478
Sub Rate:        	  8.3377% 	    20541 	  0.1334% 	       48939
Del Rate:        	  0.2322% 	      572 	  0.0653% 	       23946
Ins Rate:        	  0.1510% 	      372 	  0.0071% 	        2593
N Rate:          	  0.5707% 	     1406 	  0.1097% 	       40263


Read 2 data:      	pct reads	num reads 	pct bases	   num bases

mapped:          	 98.3647% 	   244389 	 98.8024% 	    36014811
unambiguous:     	 98.3172% 	   244271 	 98.7783% 	    36006044
ambiguous:       	  0.0475% 	      118 	  0.0241% 	        8767
low-Q discards:  	  0.0000% 	        0 	  0.0000% 	           0

perfect best site:	  2.2169% 	     5508 	  2.1948% 	      800050
semiperfect site:	  2.7184% 	     6754 	  2.7038% 	      985554
rescued:         	  0.7281% 	     1809

Match Rate:      	      NA 	       NA 	 93.5482% 	    34615871
Error Rate:      	 97.2331% 	   237627 	  6.3295% 	     2342103
Sub Rate:        	 95.5955% 	   233625 	  3.4830% 	     1288809
Del Rate:        	  7.0224% 	    17162 	  2.6712% 	      988449
Ins Rate:        	  7.3506% 	    17964 	  0.1752% 	       64845
N Rate:          	  0.8519% 	     2082 	  0.1224% 	       45286
```

As we did before we will convert the sam file to a bam file.

```bash
./bamscript2.sh
```

Part one of the metagenome assembly tutorial deals with the most of the heavy computational work that requires computers with high memory and many processors. In real applications this kind of work would be submitted as a batch job using the SLURM scheduler so that it can run without your being logged into Ceres.

The next major part of an assembly based workflow involves calling genes, annotating genes and binning contigs into genomes.  We will address this in Part 2 of the metagenomics tutorial.
