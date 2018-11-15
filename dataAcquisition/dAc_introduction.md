# Introduction to Data Acquisition

---
## What is Data Acquisition?

Data acquisition is how data is obtained through the process of downloading or transferring files from one location to another. In this section of the bioinformatics workbook, we will discuss the unix commands used to transfer/download data and also the web repositories to which data is deposited and from which data can be downloaded.

---

## What is a BioProject

[BioProjects](https://www.ncbi.nlm.nih.gov/bioproject) describe wide range of large-scale research efforts. Data are submitted to [NCBI](https://www.ncbi.nlm.nih.gov/books/n/handbook2e/glossary/def-item/ncbi/) or other [INSDC](https://www.ncbi.nlm.nih.gov/books/n/handbook2e/glossary/def-item/insdc/)-associated databases citing the BioProject accession and provide navigation between the BioProject and its datasets. The BioProject database, provides an organized framework to access information about research projects with links to data that have both been or will be deposited into archival databases maintained by members of INSDC.

## INSDC  

INSDC ( or International Nucleotide Sequence Database Collaboration) is a long-standing foundation initiative that operates between [DDBJ](http://www.ddbj.nig.ac.jp/) (DNA DataBank of Japan), [ENA](https://www.ncbi.nlm.nih.gov/books/n/handbook2e/glossary/def-item/ena/) (European Nucleotide Archive), [EMBL-EBI](https://www.ebi.ac.uk/) (European Bioinformatics Institute) and [NCBI](http://www.ncbi.nlm.nih.gov/) (National Center for Biotechnology Information). It covers the spectrum of raw data reads through alignments and assemblies to functional annotation, enriched with contextual information relating to samples and experimental configurations.


## Why Arabidopsis?
As much as possible, we will be using *Arabidopsis* data from multiple NCBI BioProject that contains datasets for many of the most common data analyses.

Arabidopsis is one of several model organisms where significant amounts of data has been collected on a wide variety of bioinformatic data analysis problems.  Additional example datasets from a variety of organism will also be provided as problem sets to explore.

## Methods to transfer files

### Scp

##### *When to use:*
 When transferring one or a few small (<100mb) files from your local computer to a remote machine.

### rsync

##### *When to use:*
 When transferring one large (>100mb) or lots of files to transfer from your local computer to a remote machine. *rsync* will compare files and directories and copy only missing or incomplete files.

##### *useful options:*
```rsync -a```: The archive option allows you to copy directories (recursively) and preserve special unix filesystem features associated with files (such as modification date, permissions, etc.)

```rsync -anv```: In addition to the archive option, adding n (dry-run) and v (verbose) allows you to preview a list of the files that would be transferred without the n option.  

### wget and curl

##### *When to use:*
When you have a URL for the file you want to transfer. For really large datasets these methods will be slower.

### SRA Toolkit
##### *When to use:* 
When you want to access any of the sequencing experiment data hosted on The NCBI Sequence Read Archive (SRA). You will need to download the SRA Toolkit in order to import data from the SRA.

### iRODS
##### *When to use:*
If you need to access data available on an iRODS system.


### FTP client

### globus










  [Table of contents](https://isugenomics.github.io/bioinformatics-workbook/)
