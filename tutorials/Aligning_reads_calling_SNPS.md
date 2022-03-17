One of the fundamental pipelines used in the 'Omics lab is that of aligning reads to a reference genome and calling [SNPs](https://www.genome.gov/genetics-glossary/Single-Nucleotide-Polymorphisms). The purpose of this page to provide an example of this pipeline. **Note:** it will need to be tailored to meet the needs of a particular project/species. This example assumes that quality control filtering of the reads has already been done.

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

Next we use [bwa](http://bio-bwa.sourceforge.net/bwa.shtml) to align the reads of reach sample back to the reference genome. Read the bwa manual for specifics, but the gist is that all forward read files are concatenated and all reverse read files are concatenated, and, since we are using paired end reads, the pairs are matched up before aligning. The names of the reads are also modified to incorporate the sample name (i.e., `-R "@RG\tID:"$sample"\tSM:"$sample""`). **Note** the use of double ("") and single (' ') quotations through out. In a bash script, inorder to unpack (read) variables, you **must** use double quotations.

Next we convert the sam file containing mapped (and unmapped) reads into a bam file with `java -jar /PATH/picard.jar SamFormatConverter` and then sort the bam file with `java -jar /PATH/picard.jar SortSam`. Then we use [samtools](http://www.htslib.org/) to index the bam file, making it easier for downstream programs to read, and to calculate some basic statistics. The final step is calculate the average depth for each sample with `/PATH/samtools depth Map.sorted.MarkDup.bam | awk '{SUM+=$3}END{print SUM/NR}' > Map.sorted.MarkDup.bam.depth.avg`. I could not get that to work as part of the for loop, due to complications of how bash reads `awk`, and so ran it manually for each one.

**Note:** In this example, I used `#!/bin/bash` and therefore have to use `bash` to run the script. Alternatively, one could use `#!/bin/sh` and use `sh` to run the script.

**Note:** `-Xmx4g` tells java to use 4 GB of memory. One may need to increase or decrease this based on the number of samples and size of the genome.

## Calling SNPs

Now that we have bam files for each sample, we can begin the process of calling SNPs. To begin, we use another for loop to make symbolic links of all the bam files in a new folder in which we will build the VCF file. A symbolic link is prefered over copying the file since it takes up less disk space. 

```bash
#!/bin/bash
string="CB_B3_B CB_B5_B CB_C3_B CB_C5_B CB_E5 IS_G5_B IS_G7_B IS_G8_B IS_H1_B PZ_C2_B PZ_D1_B PZ_D4_B PZ_D9 PZ_E3 CB_B4_B CB_B6_B CB_C4_B CB_E3  CB_E7 IS_G6  IS_G8_2 IS_G9_B IS_H2_B PZ_C7_B PZ_D2  PZ_D5  PZ_E1 PZ_E5"
for sample in $string

do

ln -s /PATH/"$sample".mapping.results/Map.sorted.MarkDup.bam /PATH/VCF/gvcf/"$sample"_Map.sorted.MarkDup.bam

ln -s /PATH/"$sample".mapping.results/Map.sorted.MarkDup.bam.bai /PATH/VCF/gvcf/"$sample"_Map.sorted.MarkDup.bam.bai

done
```
Next, we create a symbolic link to the reference genome and create a "dictionary" of the genome.

```bash
mkdir reference
ln -s /PATH/reference_genome.fasta ./
samtools faidx reference_genome.fasta
gatk CreateSequenceDictionary -R reference_genome.fasta
```

Next, we call the SNPs and produce gVCF files. For this, I originally tried to run a for loop; however, that takes a long time. Sangeet introduced me to wait scripts with which I could run multiple samples at a time, greatly speeding it up. I only provide an example of the command one might use below. With some work, one might eb able to adapt it for use in parallel, which would take the place of a wait script. A wait script has one command per line with a `&` at the end of each command and `wait` as the last command. Bash will run each command on a different node/CPU/processor.
```bash
gatk HaplotypeCaller --input "$sample"_Map.sorted.MarkDup.bam --output "$sample"_Map.sorted.MarkDup.bam.g.vcf --reference /PATH/reference_genome.fasta -mbq 20 -ERC GVCF 2> "$sample"_log.log &
```

Next we use `gatk CombineGVCFs` to combine all of the gVCF files into a single VCF file. 
```bash
gatk CombineGVCFs -R /PATH/reference_genome.fasta --variant August_Map.sorted.MarkDup.bam.g.vcf --variant September_Map.sorted.MarkDup.bam.g.vcf -O combined.vcf
```
We use `-R` to tell `gatk CombineGVCFs` what the reference genome is, we use `--variant` to call each gVCF file, and we designate the output with `-O`. Finally, we call the genotypes of each SNP.
```bash
gatk --java-options "-Xmx4g" GenotypeGVCFs -R /PATH/reference_genome.fasta -V combined.vcf -O combined_genotypes.vcf
```

## Filtering the VCF file

To ensure that the SNPs we called are true SNPs and not the result of errors in sequencing, we do some quality control filtering. The best way to do this is the let the data tell us what the cut off parameters should be. We start by creating a wait script to produce data on all the parameters we want to filter by. 
```bash
#!/bin/bash
vcftools --vcf combined_genotypes.vcf --out combined_genotypes_Q --site-quality &

vcftools --vcf combined_genotypes.vcf --outcombined_genotypes_DP --get-INFO DP &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_GQ --extract-FORMAT-info GQ &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_FS --get-INFO FS &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_MQ --get-INFO MQ &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_QD --get-INFO QD &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_missing --missing-indv &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_MQRank --get-INFO MQRankSum &

vcftools --vcf combined_genotypes.vcf --out combined_genotypes_ReadPos --get-INFO ReadPosRankSum &

wait
```
This script uses [vcftools](https://vcftools.github.io/man_latest.html) to calculate/pull the statistics/parameters from the VCF file. It can only do one parameter at a time, hence the need for the wait script.

Next, we can use an R script to plot the parameters to help us find the best cut off values.
```R 
#Plotting histograms of values that we wish to filter the VCF by

#set working directory
setwd("/PATH/")

#Load libaries
library(tidyverse)

#Load data
Q <-read.table("combined_genotypes_Q.lqual", as.is=T, header=T)

#For the GQ data, we need to convert some characters into NA so that R can see them and remove them before they cause problems.

GQ <-read.table("combined_genotypes_GQ.GQ.FORMAT", as.is=T, header=T) %>%
type_convert(, na=c("", ".", "NA"))

#Then we drop the NAs
GQ<-drop_na(GQ)

DP <-read.table("combined_genotypes_DP.INFO", as.is=T, header=T)

FS <-read.table("combined_genotypes_FS.INFO", as.is=T, header=T)

MQ <-read.table("combined_genotypes_MQ.INFO", as.is=T, header=T)

MQ_Rank <-read.table("combined_genotypes_MQRank.INFO", as.is=T, header=T)

#We need to tell R that MQ RankSum is a numeric value and not a character because it gets confused when reading it in.
MQ_Rank$MQRankSum <-as.numeric(MQ_Rank$MQRankSum)

#Then we can drop the NAs from it as well.
MQ_Rank <- drop.na(MQ_Rank)

#We need to repeat this for Pos_Rank and QD
Pos_Rank <-read.table("combined_genotypes_ReadPos.INFO", as.is=T, header=T)

Pos_Rank$ReadPosRankSum <- as.numeric(Pos_Rank$ReadPosRankSum)

Pos_Rank <- drop_na(Pos_Rank)

QD <-read.table("combined_genotypes_QD.INFO", as.is=T, header=T)

QD$QD <-as.numeric(QD$QD

QD<-drop_na(QD)

#Plots

GQ_density_plot <-ggplot(GQ) +
geom_density(aes(x=August, color="August")) +
geom_density(aes(x=September, color="September")) +
scale_color_manual(name= "Sample",
			values=c("August"= "#543005",
					"September"= "#8c510a")) +
#What we did above was specify what colors we want to be used for each value.
#If this is not desired, then one can modify and use a pre-built color scheme.
xlab("GQ") +
theme(legend.position="bottom")
guides(fill=guide_legend(ncol=6))


png("GQ_all_samples.png", width = 1000, height = 1000)
print(GQ_density_plot)
dev.off()

png("DP1000.png", width = 1000, height = 1000)
plot(density(DP$DP),
	main="DP",
	xlab="DP",
	xlim=c(0,1000))
dev.off()

png("Q100000.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 100000))
dev.off()

png("Q10000.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 10000))
dev.off()

png("Q5000.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 5000))
dev.off()

png("Q2500.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 2500))
dev.off()

png("Q1000.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 1000))
dev.off()

png("Q500.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 500))
dev.off()

png("Q200.png", width = 1000, height = 1000)
plot(density(Q$QUAL),
	main="Q",
	xlab="Q",
	xlim=c(0, 200))
dev.off()

png("FS.png", width = 1000, height = 1000)
plot(density(FS$FS),
	main="FS",
	xlab="FS",
	xlim=c(0,100))
dev.off()

png("MQ.png", width = 1000, height = 1000)
plot(density(MQ$MQ),
	main="MQ",
	xlab="MQ")
dev.off()

png("MQ_Rank.png", width = 1000, height = 1000)
plot(density(MQ_Rank$MQRankSum),
	main="MQ Rank Sum",
	xlab="MQ Rank Sum")
dev.off()

png("Pos_Rank.png", width = 1000, height = 1000)
plot(density(Pos_Rank$ReadPosRankSum),
	main="Position Rank Sum",
	xlab="Position Rank Sum")
dev.off()

png("QD.png", width = 1000, height = 1000)
plot(density(QD$QD),
	main="QD",
	xlab="QD")
dev.off()
```
To run the R script on the server, simply run:
```bash
conda activate r_env

Rscript --vanilla name.of.script.R
```

After we view the graphs and decide on cutoff values, we can then use the following script to filter the VCF file. In this script, the `&&` tells bash not to move on to the next command unless the first one finished without error. **Be sure to change the parameters to the values proper for your data!**
```bash
#!/bin/bash
gatk VariantFiltration --reference /PATH/reference_genome.fasta --variant combined_genotypes.vcf --output filtered.v1.snp.VF.vcf --filter-name "my_filter1" --filter-expression "QD < 2.0" --filter-name "my_filter2" --filter-expression "FS > 60.0" --filter-name "my_filter3" --filter-expression "MQ < 40.0" --filter-name "my_filter4" --filter-expression "MQRankSum < -10.0" --filter-name "my_filter5" --filter-expression "ReadPosRankSum < -10.0" --filter-name "my_filter6" --filter-expression "DP > 1500" --filter-name "my_filter7" --filter-expression "DP < 30" &&

gatk SelectVariants --reference /PATH/reference_genome.fasta --variant filterd.v1.snp.VF.vcf --select-type-to-include SNP --output filtered.v1.snp.filtered.vcf.gz --restrict-alleles-to BIALLELIC --exclude-filtered --exclude-non-variants &&

vcftools --gzvcf filtered.v1.snp.filtered.vcf.gz --minQ 100 --minGQ 10 --recode --recode-INFO-all --out final.v1.snp.filtered.Q100.GQ10.vcf
```
Congrats! Woot! You now have a VCF file with your SNPs!
