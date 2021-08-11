# Blast Quick Guide

You can download blast to run locally [here](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download) and NCBI has decent, albeit tr;dl, documentation in the form of two books: [Blast Help](https://www.ncbi.nlm.nih.gov/books/NBK1762/) and [Blast User Manual](https://www.ncbi.nlm.nih.gov/books/NBK279690/).

## Databases
### NCBI Database
To download an NCBI database, run `perl update_blastdb.pl --decompress [database name]`. This script can also be used to update (aka, get the latest version) of databases already downlaoded.

The nt and nr databases have been downloaded to the 'Omics lab server and are located in `/scratch/Tools/blast_db/`. However, this is a non-standard location. In order to tell the blast executables where to look, one must either run `export BLASTDB=/scratch/Tools/blast_db/` or edit one's .bash_profile to include the following:

```
BLASTDB=$BLASTDB:/scratch/Tools/blast_db/

export BLASTDB
```

### Custom databases
To build a custom database use `makeblastdb`.

```
makeblastdb -in [fasta file with your sequences] -dbtype [nucl or prot] -out [name of database]
```

## Ouput
Blast has several options for formatting the output using `-outfmt`. The different options are numbered. A complete list can be found in the [table](https://www.ncbi.nlm.nih.gov/books/NBK279684/#_appendices_Options_for_the_commandline_a_) in the appendix of the blast user manual. The most useful format for bioinformatics is the tabular format, which is option "6". This format is also customizable, which allows the user to get exactly the information they want in a format that is easy to manipulate. Some examples are provided below.

One draw back to using the tabular format is that blast does not add column names. This [wiki](https://www.metagenomics.wiki/tools/blast/blastn-output-format-6) provides the default list of columns provided in tabular format. It also provides a list of the columns one can request.

To make it easier on me, I run a variation of the following sed command to insert column names on the first line. If one asks for columns other than the default, simply edit the command to include them.

```
sed -i '1 i\query\tsubject\tpident\tlength\tmismath\tgapopen\tqstart\tqend\tsstart\tsend\tevalue\tbitscore' blast_output
```

### Getting aligned sequences

To get only the aligned sequences returned, use the [instructions kindly provided here](https://www.biostars.org/p/222341/).

Run your blastn/p search.
```
blastn -task megablast -query [your query sequence(s)] -evalue 1e-10 -db [whatever database you are using] -out [output name] -outfmt '6 std sseq'
```

Then retrieve the sequences.

```
cut -f 2,13 [blast output] | awk '{printf(">%s\n%s\n", $1, $2); }' > blastout.aligned.fasta
```

### Getting gene names or taxonomy
Tabluar format only provides the GenBank acession number by default. Many times, though, one may want the gene names or the species names. This can be accomplished by adding `stitle sscinames` to the columns reported. Int he following example, I ask for the query name, the subject name, the percent idenity, the length, the e-value, the subject title (which would include the gene name), and the scientific name of the subject.

```
blastn -task megablast -db nt -query fasta -outfmt "6 qseqid sseqid pident length evalue stitle sscinames" -max_target_seqs 5 -evalue 1e-5 -num_threads 4 -out blast_results
```

For a more thorough means of assigning taxonomy, see [here]().

## Other useful blast flags/options
Here are some flags/options that will help while running blast. The full list of flags/options is avilable in the [appendix table](https://www.ncbi.nlm.nih.gov/books/NBK279684/#_appendices_Options_for_the_commandline_a_) of the user manual.

- `-num_threads` tells the executable how many threads/cores to run on
- `-max_target_seqs` tells the executable how many hits to return per input (query) sequence 
