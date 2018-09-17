# Retrieve FASTA sequences using sequence IDs
## 1. cdbfasta/cdbyank
This is a tutorial for using file-based hashing tools (`cdbfasta` and `cdbyank`) that can be used for creating indices for quick retrieval of any particular sequences from large multi-FASTA files. Use `cdbfasta` to create the index file for a multi-FASTA file and `cdbyank` to pull records based on that index file.
To create an index file for the large multi-FASTA file
```
module load <name of cdbfasta module on your system>
cdbfasta /path/to/the/largeFASTA.fa
```
Once done, you will see a `*.cidx` binary file in the original directory (`/path/to/the/largeFASTA.fa.cidx`). Now you can retrieve any sequence from this file using `cdbyank`.

```
cdbyank -a ACCID /path/to/the/largeFASTA.fa.cidx
```
For long and complex fasta file accessions there are two options to create a shortcut
index file. This way only the first
"<db>|<accession>" pair is sufficient to retrieve a sequence:

```
 -c : the index file is built only by storing the "shortcut key"
 -C : the index file is built by storing both the "shortcut key" and the full
      keys
```

 You can retrieve multiple sequences if you have a file of their ids:
```
cat ids_file.txt | cdbyank /path/to/the/largeFASTA.fa.cidx > ids_file.fasta
```

This is much faster and cleaner than NCBI-BLAST `blastdbcmd`


## 2. NCBI blast ##

The `makeblastdb` application produces BLAST databases from FASTA files. As long as it has a specific format, you will be able to reterive the sequences using `blastdbcommand`. From the [tutorial](http://www.ncbi.nlm.nih.gov/books/NBK279688/), here is the example:
```
cat mydb.fsa
>gnl|MYDB|1 this is sequence 1
GAATTCCCGCTACAGGGGGGGCCTGAGGCACTGCAGAAAGTGGGCCTGAGCCTCGAGGATGACGGTGCTGCAGGAACCCG
TCCAGGCTGCTATATGGCAAGCACTAAACCACTATGCTTACCGAGATGCGGTTTTCCTCGCAGAACGCCTTTATGCAGAA
GTACACTCAGAAGAAGCCTTGTTTTTACTGGCAACCTGTTATTACCGCTCAGGAAAGGCATATAAAGCATATAGACTCTT
GAAAGGACACAGTTGTACTACACCGCAATGCAAATACCTGCTTGCAAAATGTTGTGTTGATCTCAGCAAGCTTGCAGAAG
GGGAACAAATCTTATCTGGTGGAGTGTTTAATAAGCAGAAAAGCCATGATGATATTGTTACTGAGTTTGGTGATTCAGCT
TGCTTTACTCTTTCATTGTTGGGACATGTATATTGCAAGACAGATCGGCTTGCCAAAGGATCAGAATGTTACCAAAAGAG
CCTTAGTTTAAATCCTTTCCTCTGGTCTCCCTTTGAATCATTATGTGAAATAGGTGAAAAGCCAGATCCTGACCAAACAT
TTAAATTCACATCTTTACAGAACTTTAGCAACTGTCTGCCCAACTCTTGCACAACACAAGTACCTAATCATAGTTTATCT
CACAGACAGCCTGAGACAGTTCTTACGGAAACACCCCAGGACACAATTGAATTAAACAGATTGAATTTAGAATCTTCCAA
>gnl|MYDB|2 this is sequence 2
GAATTCCCGCTACAGGGGGGGCCTGAGGCACTGCAGAAAGTGGGCCTGAGCCTCGAGGATGACGGTGCTGCAGGAACCCG
TCCAGGCTGCTATATGGCAAGCACTAAACCACTATGCTTACCGAGATGCGGTTTTCCTCGCAGAACGCCTTTATGCAGAA
GTACACTCAGAAGAAGCCTTGTTTTTACTGGCAACCTGTTATTACCGCTCAGGAAAGGCATATAAAGCATATAGACTCTT
GAAAGGACACAGTTGTACTACACCGCAATGCAAATACCTGCTTGCAAAATGTTGTGTTGATCTCAGCAAGCTTGCAGAAG
GGGAACAAATCTTATCTGGTGGAGTGTTTAATAAGCAGAAAAGCCATGATGATATTGTTACTGAGTTTGGTGATTCAGCT
TGCTTTACTCTTTCATTGTTGGGACATGTATATTGCAAGACAGATCGGCTTGCCAAAGGATCAGAATGTTACCAAAAGAG
CCTTAGTTTAAATCCTTTCCTCTGGTCTCCCTTTGAATCATTATGTGAAATAGGTGAAAAGCCAGATCCTGACCAAACAT
TTAAATTCACATCTTTACAGAACTTTAGCAACTGTCTGCCCAACTCTTGCACAACACAAGTACCTAATCATAGTTTATCT
CACAGACAGCCTGAGACAGTTCTTACGGAAACACCCCAGGACACAATTGAATTAAACAGATTGAATTTAGAATCTTCCAA
>gnl|MYDB|3 this is sequence 3
GAATTCCCGCTACAGGGGGGGCCTGAGGCACTGCAGAAAGTGGGCCTGAGCCTCGAGGATGACGGTGCTGCAGGAACCCG
TCCAGGCTGCTATATGGCAAGCACTAAACCACTATGCTTACCGAGATGCGGTTTTCCTCGCAGAACGCCTTTATGCAGAA
GTACACTCAGAAGAAGCCTTGTTTTTACTGGCAACCTGTTATTACCGCTCAGGAAAGGCATATAAAGCATATAGACTCTT
GAAAGGACACAGTTGTACTACACCGCAATGCAAATACCTGCTTGCAAAATGTTGTGTTGATCTCAGCAAGCTTGCAGAAG
GGGAACAAATCTTATCTGGTGGAGTGTTTAATAAGCAGAAAAGCCATGATGATATTGTTACTGAGTTTGGTGATTCAGCT
TGCTTTACTCTTTCATTGTTGGGACATGTATATTGCAAGACAGATCGGCTTGCCAAAGGATCAGAATGTTACCAAAAGAG
CCTTAGTTTAAATCCTTTCCTCTGGTCTCCCTTTGAATCATTATGTGAAATAGGTGAAAAGCCAGATCCTGACCAAACAT
TTAAATTCACATCTTTACAGAACTTTAGCAACTGTCTGCCCAACTCTTGCACAACACAAGTACCTAATCATAGTTTATCT
```

database can be generated as follows:
```
module load < name of ncbi-blast module on your system>
makeblastdb -in mydb.fsa -parse_seqids -dbtype nucl
```
The text in the definition line of each sequence will be stored in the BLAST database and displayed in the BLAST report, but it can not be used to fetch individual sequences using blastdbcmd ("this is sequence 2" in above example) .

 The `–parse_seqids` flag will enable retrieval of sequences based upon sequence identifiers. The identifier should begin right after the “>” sign on the definition line, contain no spaces, and follow the formats described [here](http://www.ncbi.nlm.nih.gov/toolkit/doc/book/ch_demo/#ch_demo.T5). The identifier consists of a two- or three-letter tag indicating the ID’s type, followed by one or more data fields, which are separated from the tag and each other by vertical bars (|).


To retrieve a sequence:
```
blastdbcmd -entry <id> -db <db name> -target_only
```
Each one of the [provided data fields](http://www.ncbi.nlm.nih.gov/toolkit/doc/book/ch_demo/#ch_demo.T5) is sufficient for querying any given sequence. Among these, the most commonly used are NCBI sequence IDs, database specific accession numbers, and its gene/protein name.

User supplied sequences should make use of the local or general identifiers. If the first token after the “>” does not contain a bar (“|”) it will be parsed as a local ID.


#### Example
```
blastdbcmd -entry "gnl|MYDB|1" -db mydb.fsa -target_only

>gnl|MYDB|1 this is sequence 1
GAATTCCCGCTACAGGGGGGGCCTGAGGCACTGCAGAAAGTGGGCCTGAGCCTCGAGGATGACGGTGCTGCAGGAACCCG
TCCAGGCTGCTATATGGCAAGCACTAAACCACTATGCTTACCGAGATGCGGTTTTCCTCGCAGAACGCCTTTATGCAGAA
GTACACTCAGAAGAAGCCTTGTTTTTACTGGCAACCTGTTATTACCGCTCAGGAAAGGCATATAAAGCATATAGACTCTT
GAAAGGACACAGTTGTACTACACCGCAATGCAAATACCTGCTTGCAAAATGTTGTGTTGATCTCAGCAAGCTTGCAGAAG
GGGAACAAATCTTATCTGGTGGAGTGTTTAATAAGCAGAAAAGCCATGATGATATTGTTACTGAGTTTGGTGATTCAGCT
TGCTTTACTCTTTCATTGTTGGGACATGTATATTGCAAGACAGATCGGCTTGCCAAAGGATCAGAATGTTACCAAAAGAG
CCTTAGTTTAAATCCTTTCCTCTGGTCTCCCTTTGAATCATTATGTGAAATAGGTGAAAAGCCAGATCCTGACCAAACAT
TTAAATTCACATCTTTACAGAACTTTAGCAACTGTCTGCCCAACTCTTGCACAACACAAGTACCTAATCATAGTTTATCT
CACAGACAGCCTGAGACAGTTCTTACGGAAACACCCCAGGACACAATTGAATTAAACAGATTGAATTTAGAATCTTCCAA
```

Note that the full identifier string (e.g., “gnl|MYDB|1” in above example) should be used to retrieve sequences with a general ID.

## 3. Bioawk ##
If all your ids are saved in `IDs.txt`, then you can use this command:
```
bioawk -cfastx 'BEGIN{while((getline k <"IDs.txt")>0)i[k]=1}{if(i[$name])print ">"$name"\n"$seq}' input.fasta
```
#### Possible problems:

* The ids in the id file contain ">" character.
* The text in the definition line is included in the id file entries.
* There is any space or tab in any id entry.

## 4. seqtk ##
This method can be applied directly to FASTA or a FASTQ file, compressed or uncompressed files. [Seqtk](https://github.com/lh3/seqtk) is a fast and lightweight tool for processing biological data (`FASTA`/`FASTQ`). if you have a list of identifiers that you would like to extract from a file, you can run this command as follows:  

```
#Extract sequences with names in file name.list, one sequence name per line:
seqtk subseq input.fasta name.list > output.fasta
```
[Table of contents](https://isugenomics.github.io/bioinformatics-workbook/)
