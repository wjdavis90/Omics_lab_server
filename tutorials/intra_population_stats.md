After one has aligned reads to a reference genome and called SNPs, we can look at some measures of diversity within each population. To do so, we will need VCF file and text files that list all the individuals that belong to a population. As an example, the text file `august.individuals`:
```bash
A1
A2
A3
A4
A5
```
Then we can use vcftools to pull those individuals out. However, we only want positions that are variable *within* populations, i.e., we do not want to count SNPS that are fixed within a population. Or, to phrase it differently, positions with 0/0 or 1/1 or 0/1 in all individuals in that population need to be removed. A good rule of thumb is to use `--maf 0.05`; however, be sure to speak with Sangeet and/or read the ltierature to be sure this is an appropriate value for your system. An example of the command to use:
```bash
vcftools --vcf final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --keep august.individuals --maf 0.05 --recode --recode-INFO-all --out august.filtered.maf.0.05
```
With these individual popualtion VCFs we can then estimate some intra-population statstics.

## pi (nucleotide diversity)

One measure of intra-population diveristy is [pi](https://en.wikipedia.org/wiki/Nucleotide_diversity), which estimates the amount of polymorphism within a population. We can use vcftools to measure this in 15kb windows across the genome. The output file has the suffix ".windowed.pi".
```bash
vcftools --vcf august.filtered.maf.0.05.recode.vcf --out august --window-pi 15000
```
We can then visulize the nucleotide diversity within a population using the [ggridges](https://github.com/wilkelab/ggridges) package of R. These are known as [joy plots](https://katherinemwood.github.io/post/joy/), and there is a good tutorial [here](https://wilkelab.org/ggridges/articles/introduction.html).
```R
#Making joy plots of pi

#load libraries
library(tidyverse)
library(patchwork)
library(RColorBrewer)
library(ggridges)

#set output resolution
ppi <- 1000


#Set working directory
setwd("/PATH/")

#Load data
august.pi.1 <-read.table("august.windowed.pi", as.is= TRUE, header=TRUE)

september.pi.1 <-read.table("september.windowed.pi", as.is= TRUE, header=TRUE)

september.pi.1 <-read.table("september.windowed.pi", as.is= TRUE, header=TRUE)

#Split, Transform, Merge Pi data

augsut.pi.2 <- august.pi.1 %>%
			subset(, select=c(CHROM,
						BIN_START,
						august_PI)) %>%
			rename(August = augsut_PI)

september.pi.2 <- september.pi.1 %>%
			subset(, select=c(CHROM,
						BIN_START,
						september_PI)) %>%
			rename(September = september_PI)

october.pi.2 <- october.pi.1 %>%
			subset(, select=c(CHROM,
						BIN_START,
						october_PI)) %>%
			rename(October = october_PI)

merge1 <-merge(augsut.pi.2, september.pi.2, all.x=TRUE, all.y=TRUE)


PI <-merge(merge1, october.pi.2, all.x=TRUE, all.y=TRUE) %>%
			unite("windows", CHROM:BIN_START, sep= ":") %>%
			pivot_longer(!windows, names_to= "Populations", values_to= "PI")


#PI joy plot

PI_joy_plot <-ggplot(PI,
				aes(x=PI,
				y=Islands,
				fill=Populations)) +
geom_density_ridges(scale=0.9) +
scale_fill_brewer(palette="Paired")

tiff("PI_joy_plot.tiff",
width= 7.4*ppi,
height=9.4*ppi,
res = ppi,
compression= "lzw")
print(PI_joy_plot)
dev.off()
```

One may also want the average of a population, and for that we need a simple `awk` script.
```bash
awk '{ sum += $5; n++ } END { if (n > 0) print sum / n; }' august.windowed.pi
```
This will output the average pi to the terminals creen, if you want it in a file you will need to redirect it with `>`.

## snp density


## Tajima's D

## Sequence depth

## Linkage Disqeuilibrium

## Effective popualtion size
