# Introduction to MetaGenomics

## Caveats of cultivation-based sampling

Many microbes live in communities whose members interact and communicate in complex ways. Microbial communities often interact through the medium (water or soil) in which they grow, exchanging nutrients, biochemical products, and chemical signals without direct cell-to-cell contact. Some are in physical contact with others of their own kind and with other species.
In the pure-culture model, the presence of multiple species in the same culture medium means “contamination,”. This is one of the reasons that
most microbes are extremely difficult to culture in the lab (about 1% from the environmental samples and 35% from human gut which is a less complex environment).

## MetaGenomics

MetaGenomics is the study of genetics from the material directly obtained from environmental samples without the use of cultivation-based methods. Recent studies use either "shotgun" or PCR directed sequencing to get largely unbiased samples of all genes from all the members of the sampled communities.

## MetaGenomics steps

* Planning
* Sampling
* Extraction of DNA
* Sequencing
* Assembly
* Annotation
* Statistical analysis

### Extraction of DNA

* If the target community is associated with a host then either fractionation or selective lysis may be required to minimize the masking effect of the large genome of the host.
* The extracted DNA should be representative of all cells present
* Extracted DNA should be high quality and enough for sampling and making libraries. Longer pieces of DNA are preferred.

### Sequencing

#### Classical method
[Sanger](https://www.ncbi.nlm.nih.gov/pubmed/271968) is the classical sequencing method since 1977. In Sanger sequencing, chain-terminating dideoxynucleotides (ddNT) are incorporated into the growing DNA chain at random positions. The sequence is then inferred from the set of incrementally terminated DNA chains separated by polyacrylamide gel electrophoresis.

Challenges of this method are:
* Poor quality at the first 15-40 bases due to primer binding. The quality increasingly drops after 700-900 bases.
* In cases that the DNA fragments are cloned before sequencing, the resulting sequence may contain part of cloning vector.
* After 1000 base pair, the method can not separate DNA fragments that differ in length by only one nucleotide  
* Expensive  compare to other methods

For more information on the method visit [sanger sequencing on wikipedia.](https://en.wikipedia.org/wiki/Sanger_sequencing)

#### Next generation methods
Improvements from the first generation methods:
* Instead of requiring bacterial cloning of DNA fragments they rely on the preparation of NGS libraries in a cell free system.
*  Instead of hundreds, thousands-to-many-millions of sequencing reactions are produced in parallel. *

* The sequencing output is directly detected without the need for electrophoresis; base interrogation is performed cyclically and in parallel.

#### Single vs. pair Reads

Second generation sequencing technology begins with breaking up a long piece of DNA into lots of short pieces of DNA called "fragments". The resulting set of DNA fragments are called a "library". Each fragment is then sequenced individually and in parallel. There are two ways of sequencing a fragment:
* single read sequencing : if only one one end is sequenced.
* Pair read sequencing : if the technology can sequence both ends of a fragment. (standard practice on Illumina instruments like GaIIx, HiSeq and MiSeq)

 for pair-end reads you want to make sure that fragment length (F) is at least double the size of the read length (L). Typical Illumina fragment libraries would use F ~ 450 bp. As reads getting longer ( for example > 250 on  MiSeq) the standard library size gets small and pair end reads will overlap. Moreover, the new enzymatic libraries contains a wide range of F sizes ( for example in Nextera library F ranges from 100bp to 900bp).


### Illumina sequencing


#### Illumina sequence identifiers  

###### Example:

```@MISEQ05:522:000000000-ALD3F:1:1101:22198:19270 1:N:0:ATGTC
```

|` @MISEQ05` | instrument name |
| --- | --- |
| `522` | run id |
| `000000000-ALD3F`| the flowcell id|
| `1` | flowcell lane |
| `1101`| tile number within the flowcell lane|
|`22198`| x coordinate of the cluster within the tile|
|`19270`| y coordinate of the cluster within the tile|
|`1`| the member of a pair, 1 or 2 (paired-end or mate-pair only)|
|`N`| Y if the read is filtered, N otherwise|
|`0`| 0 when none of the control bits are on, otherwise it is an even number|
| `ATGTC`| index sequence|
