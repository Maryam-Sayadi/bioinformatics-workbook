# Converting FASTQ format to FASTA

There are several ways to convert fastq to fasta sequences. Some methods are listed below:

### Using SED
`sed` can be used to selectively print the desired lines from a file, so if you print the first and 2rd line of every 4 lines, you get the sequence header and sequence needed for fasta format.
```
sed -n '1~4s/^@/>/p;2~4p' INFILE.fastq > OUTFILE.fasta
```

Please note that there are peculiar difference between sed under UNIX and sed under BSD LINUX in Mac OS. So for Mac users there are two options:

* Mac user can search for the equivalent of the sed commands under Mac if they get "invalid command code" error message.
* Other option is to install gnu-sed :
```
brew install gnu-sed --default-names
```
With --with-default-names option it installs sed to /usr/local/bin/.

to test if the gnu-sed version is being used try:

```
echo a | sed ’s_A_X_i’
```
which under BSD LINUX in Mac OS returns bad flag error.


###  Using PASTE
You can linerize every 4 lines in a tabular format and print first and second field using `paste`
```
cat INFILE.fastq | paste - - - - |cut -f 1,2| sed 's/@/>/'g | tr -s "/t" "/n" > OUTFILE.fasta
```

#### Possible problems using PASTE
Adding any space in cut -f command will cause an error message:

```
cat INFILE.fastq | paste - - - - |cut -f 1, 2| sed 's/^@/>/'g | tr -s "/t" "/n" > OUTFILE.fasta
```
Error message:
```
cut: [-cf] list: values may not include zero
```


###  EMBOSS: seqret
 EMBOSS is "The European Molecular Biology Open Software Suite". It can be installed and used at the command line or it can be accessed from the [web](https://www.ebi.ac.uk/Tools/sfc/emboss_seqret/).

 EMBOSS can be used for :

* Sequence alignment,
* Rapid database searching with sequence patterns,
* Protein motif identification, including domain analysis,
* Nucleotide sequence pattern analysis---for example to identify CpG islands or repeats,
* Codon usage analysis for small genomes,
* Rapid identification of sequence patterns in large scale sequence sets,
* Presentation tools for publication,

and

 * fastq-fasta conversion

After installing EMBOSS, the seqret command can be used directly in the command line:
```
seqret -sequence reads.fastq -outseq reads.fasta
```

For more information on how to install EMBOSS visit [EMBOSS home page.](http://emboss.sourceforge.net/docs/adminguide/admin.html)


###  Using AWK

```
cat infile.fq | awk '{if(NR%4==1) {printf(">%s\n",substr($0,2));} else if(NR%4==2) print;}' > file.fa
```


###  FASTX-toolkit
`fastq_to_fasta` is available in the FASTX-toolkit that scales really well with the huge datasets. The toolkit is available to use online through [Galaxy.](http://hannonlab.cshl.edu/fastx_toolkit/galaxy.html)
It can also be used at [command line](http://hannonlab.cshl.edu/fastx_toolkit/commandline.html):
```
fastq_to_fasta -h
usage: fastq_to_fasta [-h] [-r] [-n] [-v] [-z] [-i INFILE] [-o OUTFILE]
# Remember to use -Q33 for illumina reads!
version 0.0.6
	   [-h]         = This helpful help screen.
	   [-r]         = Rename sequence identifiers to numbers.
	   [-n]         = keep sequences with unknown (N) nucleotides.
			       Default is to discard such sequences.
	   [-v]         = Verbose - report number of sequences.
			       If [-o] is specified,  report will be printed to STDOUT.
			       If [-o] is not specified (and output goes to STDOUT),
			       report will be printed to STDERR.
	   [-z]         = Compress output with GZIP.
	   [-i INFILE]  = FASTA/Q input file. default is STDIN.
	   [-o OUTFILE] = FASTA output file. default is STDOUT.
```
Mac users could Possibly face some   difficulties during installation of FASTX specially on MAC OS 10.9 or older. In this case it is possible to use the [pre-compiled binary files](http://hannonlab.cshl.edu/fastx_toolkit/download.html).

For other available tools visit [FASTX-Toolkit.](http://hannonlab.cshl.edu/fastx_toolkit/)
###  Bioawk
Another option to convert fastq to fasta format using `bioawk`
```
bioawk -c fastx '{print ">"$name"\n"$seq}' input.fastq > output.fasta
```

###  Seqtk
From the same developer, there is another option using a tool called `seqtk`
```
Usage:   seqtk <command> <arguments>
Version: 1.0-r77-dirty

Command: seq       common transformation of FASTA/Q
         comp      get the nucleotide composition of FASTA/Q
         sample    subsample sequences
         subseq    extract subsequences from FASTA/Q
         fqchk     fastq QC (base/quality summary)
         mergepe   interleave two PE FASTA/Q files
         trimfq    trim FASTQ using the Phred algorithm

         hety      regional heterozygosity
         mutfa     point mutate FASTA at specified positions
         mergefa   merge two FASTA/Q files
         dropse    drop unpaired from interleaved PE FASTA/Q
         rename    rename sequence names
         randbase  choose a random base from hets
         cutN      cut sequence at long N
         listhet   extract the position of each het
```
Example:
```
seqtk seq -a input.fastq > output.fasta
```
Note that you can use either compressed or uncompressed files for this tool
## More information
*  [Introduction to Bioawk](/Appendix/bioawk-basics.md)

[Table of contents](https://isugenomics.github.io/bioinformatics-workbook/)
