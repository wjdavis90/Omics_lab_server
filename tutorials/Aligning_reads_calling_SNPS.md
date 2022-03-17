One of the fundamental pipelines used int he 'Omics lab is that of aligning reads to a reference genome and calling scripts. The purpose of this page to provide an example workflow of this pipeline. Note, however, that it may need to be tailored to meet the needs of a particular project/species. This example assumes that quality control filtering of the reads has already been done.

## Aligning reads
The first step is to align the reads to a reference genome. When I (William Davis) did this for the *Philornis downsii* project, I used a [for loop](https://swcarpentry.github.io/shell-novice/05-loop/index.html) to cycle through each population. I am going to use that script as the example below. However, with some work, I think one could adapt it for use with [parallel](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/parallel_examples.md), which would speed it up some. I have the whole script below and then I deconstruct it line by line (ignoring the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) at the top). Sample details have been omitted.
```bash
#!/bin/bash
string="space delimited list of samples"
for sample in $string

do

mkdir "$sample".mapping.results

cd "$sample".mapping.results || exit

/PATH/bwa mem -t 12 -R "@RG\tID:"$sample"\tSM:"$sample"" /PATH/reference_genome.fasta <(cat /PATH/"$sample"/*_L*_1.fq.gz) <(cat /PATH/"$sample"/*_L*_2.fq.gz) > Map.sam 2> map_error.log

java -jar /PATH/picard.jar SamFormatConverter INPUT=Map.sam OUTPUT=Map.bam VALIDATION_STRINGENCY=SILENT 2> format_converter_error.log


java -jar /PATH/picard.jar SortSam INPUT=Map.bam OUTPUT=Map.sorted.bam SORT_ORDER=coordinate VALIDATION_STRINGENCY=SILENT 2> sort_error.log


java -Xmx4g -jar /PATH/picard.jar MarkDuplicates INPUT=Map.sorted.bam OUTPUT=Map.sorted.MarkDup.bam METRICS_FILE=DupMetrics ASSUME_SORTED=true VALIDATION_STRINGENCY=SILENT MAX_FILE_HANDLES_FOR_READ_ENDS_MAP=1000 2> mark_dups_error.log


/PATH/samtools-1.10/bin/samtools index Map.sorted.MarkDup.bam 2> samtools_index.log

/PATH/samtools-1.10/bin/samtools flagstat Map.sorted.MarkDup.bam > Map.sorted.MarkDup.bam.flagstat 2> samtools_flagstat_error.log

/PATH/samtools-1.10/bin/samtools depth Map.sorted.MarkDup.bam | awk "{SUM+=$3}END{print SUM/NR}" > Map.sorted.MarkDup.bam.depth.avg

cd ../

done
```
For the above script, we first create a variable called `string` that is simple a space delimited list of the population samples (or at least their names). Then we begin the [for loop](https://swcarpentry.github.io/shell-novice/05-loop/index.html). We tell bash that we want to run the following set of commands for each sample in the variable `string`; here "sample" also becomes a varaible. Variables in for loops are denoted with the `$`. 

In our example, the list of commands to be run starts with `mkdir "$sample".mapping.results`. We are telling bash to make a directory named `"$sample".mapping.results`. The double quotes ("") help bash identify `$sample` as a varaible, and it represents each name in the list we provided. Thus, if the first sample is named "August", then it will make the firectory "August.mapping.results". The next command is to move into the directory just created. I set the script up this way because then I could use the same file names over again in the next commands, which just makes it easier to keep inputs and outputs straight.

Next we use [bwa](http://bio-bwa.sourceforge.net/bwa.shtml)
