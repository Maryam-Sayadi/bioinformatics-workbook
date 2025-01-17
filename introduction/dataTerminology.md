# Data Terminology

## Learning Objective
Upon completion of this section the learner will know the definitions for the following terms.

* base/nucleotide
* read
* contig
* scaffold
* chromosome

## What is a base?
There are four common bases in DNA sequence, ```A```denine, ```G```uanine, ```C```ytosine and ```T```hymine. ```U```racil is found in RNA in place of Thyamine

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e2/Nucleotides_1.svg/1320px-Nucleotides_1.svg.png)
Image taken from [wikipedia](https://en.wikipedia.org/wiki/Nucleotide) where more information about nucleotides can be found also be found.

## What is a read?
A read is a string of bases represented by their one letter codes. Here is an example of a read that is 50 bases long.
``TTAACCTTGGTTTTGAACTTGAACACTTAGGGGATTGAAGATTCAACAACCCTAAAGCTTGGGGTAAAAC``  



## What is a contig?

A contig is the consensus sequence generated by aligning reads to themselves.  

![](assets/contig.png)



The last line is the consensus of the aligned reads which we call a contig.

## What is a scaffold?
A scaffold is a set of contigs that have been ordered and oriented based on mate pair or long distance information.

```contig```NNNNNNNNNNNN```gitnoc```NNNNNNNN```contig```NNNNNNNN```contig```NNNN```gitnoc```

In the line above
* ```contig``` is a string of of bases (ATC or G)
* N is an unknown base
* ```gitnoc``` is the reverse complement of a contig  

## What is a chromosome?
Chromosomes are the largest DNA molecules in the cell. Scaffolds can be ordered and oriented using a genetic map or HiC data into linkage groups or chromosomes.  The ultimate goal of a genome assembly project is to assemble reads into chromosomes.

[Table of contents](https://isugenomics.github.io/bioinformatics-workbook/)
