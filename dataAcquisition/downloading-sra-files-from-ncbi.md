### File transfer with sratoolkit
[SRA (Sequence Read Archive) toolkit](https://www.ncbi.nlm.nih.gov/sra/docs/) has been configured to connect to NCBI SRA and download via FTP. The simple command to fetch a SRA file and to split it to forward/reverse reads, use this command:

<pre>
module load sratoolkit
fastq-dump --split-files --origfmt --gzip SRR000001
</pre>

You will see 4 files (SRR000001_1.fastq.gz, ...,  SRR000001_4.fastq) downloaded directly from NCBI. If the file size is more than 1Gb, submit this within a PBS script.

## File transfer with ASPERA

[Aspera Connect](https://www.ncbi.nlm.nih.gov/books/NBK242625/) is a software for uploading and downloading large data for popular browsers. It also has a command line tool (ascp) and is free to exchange data with NCBI.
File transfer will be fast (very useful if you have large number of files to download) and if a file transfer fails, it resumes automatically.

#### Aspera Connect plugin

The plugin should be downloaded and installed from [here](http://downloads.asperasoft.com/connect2/). The data could be downloaded directly from [NCBI](https://www.ncbi.nlm.nih.gov/home/download/).

Please make sure to check the firewall requirements. Mac OS X 10.11 and higher has security policies that may break aspera's function. [Extra steps](https://support.asperasoft.com/hc/en-us/articles/216125928-Workaround-to-use-Aspera-transfer-servers-on-Mac-OS-X-10-11-El-Capitan-or-Later) are required to fix this problem.

#### ascp command line

ascp will be downloaded along the Aspera Connect plugin download:


<pre>
ascp -i <asperaweb_id_dsa.openssh with path> -k 1 -T -l 200m \
anonftp@ftp-trace.ncbi.nlm.nih.gov:/<files> <local destination>
</pre>

     -i fully qualified path and file name where this pubic key file is located.
     -T to disable encryption
     -k 1 enables resume of partial transfers
     -r recursive copy
     -l max bandwidth of request(100M and higher)
     -q quiet mode (to disable progress display)

     <files> names of the files to transfer including path
     <local destination> location to store data

Note that the above command should be in single line.

for more information on the options use:

<pre>
ascp -h
</pre>

###### Some useful locations

* public key file: This file is a part of Aspera Connect distribution and is usually located in the '/etc' directory.

* Default executable location:

     windows : "C:\Program Files\Aspera\Aspera Connect\bin\ascp.exe"

    OS X:  "/Applications/Aspera Connect.app/Contents/Resources/ascp"

    Linux: " /opt/aspera/bin/ascp"


If you have large number of files to download (usually organized as <blockcode>project</blockcode>  with a SRR project number), you can save all the IDs in a file and loop through the lines.
For more information visit [Aspera Transfer Guide by NCBI](https://www.ncbi.nlm.nih.gov/books/NBK242625/#Aspera_Transfer_Guide_BK.Requirements) and [aspera command line web page.](https://support.asperasoft.com/hc/en-us/articles/216125898-Downloading-data-from-NCBI-via-the-command-line)

## wget

Finally, if you want to download using <blockcode>wget</blockcode> (which will be very slow), you can use this template:
<pre>
wget http://trace.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?cmd=dload&run_list=SRRXXXXXX&format=fastq
</pre>

If you have a previously downloaded FASTQ file, without using the <blockcode>--origfmt</blockcode>, option, then the first field (before the instrument name) is SRA id number:

##### Example


```
head -n 12 SRR447882_1.fastq

@SRR447882.1.1 HWI-EAS313_0001:7:1:6:844 length=84
ATTGATCATCGACCAGAGNCTCATACACCTCACCCCACATATGTTTCCTTGCCATAGATCACATTCTTGNNNNNNNGGTGGANA
+SRR447882.1.1 HWI-EAS313_0001:7:1:6:844 length=84
BBBBBB;BB?;>7;?<?B#AA3@CBAA?@BAA@)=6ABBBBB?ACA;0A=257?A7+;;&########################
@SRR447882.2.1 HWI-EAS313_0001:7:1:6:730 length=84
AGTTGATTGTGATATAGGNGTCTATCGACATTGATGCATAGGTCCTCTATTAAACTTGTTTTGTGATGTNNNNNNNTTTTTTNA
+SRR447882.2.1 HWI-EAS313_0001:7:1:6:730 length=84
A?@B:@CA:=?BCBC:2C#7>BACB??@4@B@<=>;'>@>3:86>=6@=B@B<;)@@###########################
@SRR447882.3.1 HWI-EAS313_0001:7:1:6:1343 length=84
CATCAATGCAAGGATTGTNCCATTGGTAACAATTCCACTCCTAACTTGTCAATTGATTTTCATATAACTNNNNNNNCCAAAANT
+SRR447882.3.1 HWI-EAS313_0001:7:1:6:1343 length=84
BCB@BBC+5BCA>BABBA#@4BCCA>?CBBB4CB(*ABB?ABBAACCB8ABBB?(<<B?:########################

```


Although. most of the programs won't mind, some of them such as   [Khmer](http://khmer.readthedocs.org/en/v1.0) might throw an error.

To remove this use the following script:

<pre>
for f in SRR447882_[12]_paired.fq; do\
awk '$1 ~ /@SRR447882*/ {$1="@"}{print}' $f | sed 's/^@ /@/g' | \
sed 's/^+SRR447882.\+/+/g' > $f.cleaned; \
done
</pre>

<p>Now the sequences should look like these:</p>

    @HWI-EAS313_0001:7:1:6:844 length=84
    ATTGATCATCGACCAGAGNCTCATACACCTCACCCCACATATGTTTCCTTGCCATAGATCACATTCTTGNNNNNNNGGTGGANA
    +
    BBBBBB;BB?;>7;?<?B#AA3@CBAA?@BAA@)=6ABBBBB?ACA;0A=257?A7+;;&########################
    @HWI-EAS313_0001:7:1:6:730 length=84
    AGTTGATTGTGATATAGGNGTCTATCGACATTGATGCATAGGTCCTCTATTAAACTTGTTTTGTGATGTNNNNNNNTTTTTTNA
    +
    A?@B:@CA:=?BCBC:2C#7>BACB??@4@B@<=>;'>@>3:86>=6@=B@B<;)@@###########################
    @HWI-EAS313_0001:7:1:6:1343 length=84
    CATCAATGCAAGGATTGTNCCATTGGTAACAATTCCACTCCTAACTTGTCAATTGATTTTCATATAACTNNNNNNNCCAAAANT
    +
    BCB@BBC+5BCA>BABBA#@4BCCA>?CBBB4CB(*ABB?ABBAACCB8ABBB?(<<B?:########################


Instead of this, you can also redonwload the original SRA file using <blockcode>--origfmt</blockcode> option, if it saves time.

<h2>Download all SRR files related to a project </h2>

If you have large number of SRR files to donwload, see if they belong to a specific project. Eg., project [[http://www.ncbi.nlm.nih.gov/Traces/sra/?study=SRP011907 | SRP011907 ]] has 283 SRR files. You can use aspera to download all 283 files at once.

Here are the steps:

  - Get the FTP link: go to [[http://www.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?view=download_reads | SRA download reads]] and look for the project id (eg., SRP011907, is located under <blockcode> reads --> ByStudy --> sra --> srp --> SRP011 --> SRP011907 </blockcode>. Once you reach the link, clicking the "save icon" (next to the total size) will give the ftp link.
  - Now, download it using Aspera as expalined before.

<pre>
module load aspera/3.3.3.81344
ascp -i $KEY/asperaweb_id_dsa.openssh -k 1 -QT -l 200m \
  anonftp@ftp-trace.ncbi.nlm.nih.gov:/sra/sra-instant/reads/ByStudy/sra/SRP/SRP011/SRP011907 \
  ./Destination_dir
</pre>
