# Megahit

###  contigs output for Megahit:

The FASTA file `final.contigs.fa` (or *OUTPUT_PREFIX.contigs.fa* if `--out-prefix is` specified) is the final result of the assembly.

* Note:  *Final.contigs.fa* has filtered out some short contigs.

For files in the folder intermediate_contigs:

* `kK.contigs.fa` contains the contigs assembled from the de Bruijn graph of order-*K*, they can be converted to a SPAdes-like FASTG file for visualiztion
* `kK.addi.fa` contains the contigs assembled after iteratively removing local low coverage unitigs in the de Bruijn graph of order-*K*
* `kK.local.fa` contains the locally assembled contigs for k=*K*
* `kK.final.contigs.fa` contains the stand-alone contigs for k=*K*; if local assembly is turned on, the file will be empty

### Visualizing MEGAHIT's contig graph in [Bandage](https://rrwick.github.io/Bandage/)

This feature is supported in version 0.3.0 or later. First you need to convert `kK.contigs.fa` file into SPADes-like FASTG format:

`megahit_toolkit contig2fastg 99 k99.contigs.fa > k99.fastg`

Now the FASTG file *k99.fastg* can be loaded into bandage.
