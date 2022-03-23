Measures of relatedness try to measure how genetically similar or disimilar members of a population are to each other. There are a few ways to go about doing this, but I have only used two.

## Unadjusted Ajk statistic

[Vcftools](https://vcftools.github.io/man_latest.html) allows one to calculate the unadjusted Ajk statistic as a measure of relatedness using the `--relatedness` flag. The unadjusted Ajk statistic was developed by [Yang et al. 2010](https://doi.org/10.1038/ng.608). The statistic ranges between 0 (completely unrelated) and 1 (completely related). Specifically, an individual chould only score a 1 when compared to themselves (assuming the species does not reproduce clonally). To estimate this statistic we simply need the VCF file and the following command. Vcftools will append `.relatedness` to the output file.
```bash
vcftools --vcf final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --out final.v1.snp.filtered.Q100.GQ10.vcf --relatedness
```
The output will be a file with the header `INDV1	INDV2	RELATEDNESS_AJK`. From this output we would like to map a pretty heatmap in R. However, the script that I used to do this, which I found as part of this [tutorial](https://plantgenomics.univie.ac.at/radseq/snpfiltering/), requires that the input be a matrix. I converted the output into a matrix by hand in Excel. There are probably better ways of doing so using the command line, and if you figure that out, please update this page with that information.

After converting the vcftools output into a matix, I used the following R script to make it into a heat map.
```R
#Set working directory
setwd("/PATH/")

#Load libraries
library(vioplot)
library(gplots)
library(RColorBrewer)

#set output resolution
ppi <- 1000


#read in data

relat <- as.matrix(read.table("final.v1.snp.filtered.Q100.GQ10.vcf.relatedness.matrix.txt", header=T))

heatmap.2(relat,
	trace="none",
	cexRow=0.6,
	cexCol=0.6,
	col= colorRampPalette(brewer.pal(8, "Blues"))(29))


tiff("relatedness_heatmap.tiff",
width= 7.4*ppi,
height=4.7*ppi,
res = ppi,
compression= "lzw")
print(heatmap.2(relat,
	trace="none",
	cexRow=0.6,
	cexCol = 0.6,
	col= colorRampPalette(brewer.pal(8, "Blues"))(29))
)
dev.off()

png("relatedness_heatmap.png")
print(heatmap.2(relat,
	trace="none",
	cexRow=0.6,
	cexCol = 0.6,
	col= colorRampPalette(brewer.pal(8, "Blues"))(29))
)
dev.off()
```

## Pairwise Identity-by-state distance

I prefer the heatmap, but Sangeet likes an ordination plot created by [plink](https://zzz.bwh.harvard.edu/plink/strat.shtml). Plink uses the pairwise identity-by-state distance (IBS) to measure relatedness amoung individuals. To begin, we need to convert the vcf file into plink format, which we can do using vcftools.
```bash
vcftools --vcf final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --out for_plink --plink
```
Then we can use plink to estimate the statistic. First, we have to create a `plink.genome` file.
```bash
plink --noweb --file for_plink --genome
```
Then we can use plnik to calculate the IBS and cluster the individuals accordingly.
```bash
plink --noweb --file for_plink --read-genome plink.genome --cluster --mds-plot 4
```
For both commands, we use `--noweb` to tell it to skip connecting to the internet and checking for updates. 

With the output files, I used the following R script to make the ordination plot. I used `mutate` to create a new column called "Population". I did this so that I could use different shapes and different colors to represent each population so that the clusters stand out more clearly. 
```R 
#Relatedness mds from plink

#set working directory
setwd("C://Users//wjdavis//Documents//R_working_dir//Philornis_project//relatedness")

#set output resolution
ppi <- 1000

#Load libraries
library(tidyverse)
library(RColorBrewer)

#read in data

mds<-read.table("plink.mds", header=TRUE)

mds <- mds %>%
	mutate(Population = case_when(
					FID %in% c("A1",
                      "A2",
                      "A3",
                      "A4",
                      "A5") ~ "August",

					FID %in% c("S1",
                      "S2",
                      "S3",
                      "S4",
                      "S5") ~ "September",

					FID %in% c("O1",
                      "O2",
                      "O3",
                      "O4",
                      "O5") ~ "October"))


mds_plot <- ggplot(mds,
	aes(x=C1,
		y=C2,
		color=Population,
		shape=Population)) +
scale_color_brewer(palette="Dark2") +
geom_point(size=3) +
theme(legend.position="bottom")

tiff("relatedness_mds_plot.tiff",
width= 7.4*ppi,
height=4.7*ppi,
res = ppi,
compression= "lzw")
print(mds_plot)
dev.off()

png("relatedness_mds_plot.png")
print(mds_plot)
dev.off()
```
