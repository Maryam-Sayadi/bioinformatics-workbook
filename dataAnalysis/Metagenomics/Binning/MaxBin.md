 # MaxBin

### Introduction

[MaxBin](https://downloads.jbei.org/data/microbial_communities/MaxBin/README.txt) is a software that is capable of clustering metagenomic contigs into different bins, each consist of contigs from one species. MaxBin uses the nucleotide composition information and contig abundance information to do achieve binning through an Expectation-Maximization algorithm.

### Installation
 1. Download MaxBin and unzip it
 2. Enter src directory under MaxBin and **make** it.
 3. Run `./autobuild_auxiliary` at MaxBin directory to download, compile and set up the auxiliary software package
 4. *MaxBin* should be ready to go.


 ### Run MaxBin
Run `run_MaxBin.pl`. Here are the options:

* contig : contig file name (required)
* out : output file header ( required)
* At least one of the following parameters is needed:
 *  thread :number of threads
 * probe_threshhold : minimum probability for EM algorithm;default 0.8
 * plotmaker : specify this option if you want to plot the markers in each contig. **R** must be installed for this option to work.
 * verbose : output log. *warning*: log will be very long!
 * makerset : choose between 107 marker genes by default or 40 marker genes. ( See *marker gene note* for more information)

### Marker Gene note

By default MaxBin will look for 107 marker genes present in >95% of bacteria. Alternatively you can also choose 40 marker gene sets that are universal among bacteria and archaea ([Wu *et. al.* ](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0077033)). This option may be better suited for environment dominated by archaea; however it tend to split genomes into more bins. You can choose between different marker gene sets and see which one works better.

### Abundance
The contig abundance information can be provided in two ways: user can choose to provide the abundance file or MaxBin will use *Bowtie2* to map the sequencing reads against contigs and generate the abundance information. In the second case the reads file ( in FASTA format) should be specified in ` -reads`.

* Note: If you are providing the abundances make sure to follow this format:
`(contig header) \t (abundance)`. (\t stands for tab)


### Providing more than one reads or abundance files
The reads and abundance files can be provided via a file consisting of all reads or abundance files.
you can feed all information into MaxBin using `-abund_list` and `-reads_list` options. Handy for a large number of reads or abundance files.


### MaxBin Output

Assuming the output header is (out), here are the output files:

* **(out).0XX.fasta** : the *XX* bin.  *e.g.*
 `out.001.fasta`
* **(out).summary** : a summary file describing which contigs are being classified into which bin.
* **(out).log**
* **(out).marker** : marker gene presence number for each bin. This table is ready to be plotted by R.
* **(out).marker.pdf** : visualization of the marker gene presence number using R. will only appear if `-PLOTMAKRKER` is specified.
* **(out).noclass** : this file stores all sequences that pass the minimum length threshold but are not classified successfully.
* **(out).tooshort** : this file stores all sequences that do not meet the minimum length threshold.
* **(out).marker_of_each_gene.tar.gz** : this tarball file stores all markers predicted frm the individual bins. Use ` tar -zxvf (out).marker_of_each_gene.tar.gz`  to extract the markers.


#### Example:
