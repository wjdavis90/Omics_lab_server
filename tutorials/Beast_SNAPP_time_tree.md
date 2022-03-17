Sometimes we will want a divergence time tree of our populations. This page describes one way of doing so based on [Mikkelsen et al. 20XX](), who based it on a [tutorial](https://github.com/elsemikk/tutorials/blob/master/divergence_time_estimation/README.md) and [Stange et al. 2018](https://doi.org/10.1093/sysbio/syy006). This method uses constraints to calibrate the age of the nodes. We will be using [SNAPP](https://www.beast2.org/snapp/) within Beast2 to infer the trees and the node ages from a VCF file. SNAPP can only handle biallelic data dna it cannot handle missing data; so, our first step is to filter the VCF file accordingly. Also, we will thin the VCF file in part to account for linkage and in part to reduce computational time. As well, as per [Stange et al. 2018](https://doi.org/10.1093/sysbio/syy006), increasing the number of individuals representing the each popualtion increases the computational time but does not affect the accuracy of the inferred tree or node ages. Thus, we will also reduce the number of individuals in each population to 1. For the purposes of this example, we have three populations: August, September, and October. There are 5 individuals in each population numbered 1 to 5, e.g., A1, A2, A3, etc. After we filter the VCF file, we will use a custom script by Stange et al. 2018 to prepare an input xml file for Beast2. Then we will run Beast2. Finally, we will process and visualize the results.

## Filtering the VCF file
First, we will filter the VCF then thin it. For birds, 5000 is a a good value for thinning, but be sure to figure out the most appropriate number for your populations.
```bash
vcftools --vcf final.v1.snp.filtered.Q100.GQ10.vcf --out v3.snp.filtered.biallelic.no.missing --max-missing 1.0 --min-alleles 2 --max-alleles 2 --recode

vcftools --vcf v3.snp.filtered.biallelic.no.missing.recode.vcf --out SNAPP.v4.snp.filtered.biallelic.no.missing.5kthinned --thin 5000 --recode
```

Next, we will reduce the VCF to be 1 individual per population. There are many ways one can go about doing this, but here is one way. First, we number the individuals in each population. 
```bash
August
A1
A2
A3
A4
A5

September
S1
S2
S3
S4
S5

October
O1
O2
O3
O4
O5
```
Then we ask [random.org](https://www.random.org/integers/) to producea set of 3 random integers between 1 and 5.
```bash
Random Integer Generator

Here are your random numbers:

5	2	2	

Timestamp: 2022-03-17 16:16:37 UTC

Note: The numbers are generated left to right, i.e., across columns.
```
Thus, We only include individuals A5, S2, and O2 in the Beast2 run. We use vcftools to pull out those individuals. 
```bash
vcftools --vcf SNAPP.v4.snp.filtered.biallelic.no.missing.5kthinned.recode.vcf --out SNAPP.v5.snp.filtered.biallelic.no.missing.5kthinned.individual.filtered --indv A5 --indv S2 --indv O2 --recode
```

## Preparin the xml file

To start, we need a list of samples/individuals formatted like this.
```
species specimen
A5	A5
S2	S2
O2	O2
```
**NOTE:** The "species" column cannot contain names such as "A" or "B" as Beast2 will read these names as internal variables and get confused. 

Next, we need an estimate of divergence date to use as a constraint on the tree. This esitmate can come from previous studies (e.g., [Bao et al. 2010](https://doi.org/10.1016/j.ympev.2010.03.038)) or fossil evidence or [Time Tree](http://www.timetree.org/). For the purposes of this example, we are going to use Time Tree to find out that the estimated divergence time of our clade is 4.37 mya. Note: [Stange et al. 2018](https://doi.org/10.1093/sysbio/syy006) found that root age contraints yielded slightly better results, i.e., slightly more accurate and precise (smaller 95% highest posterior densities (HPD). Thus, it is reccommended to contrain the root of the tree. 

With our estimated divergence time in hand, we can write an input file called `constraints.txt`. 
```bash
lognormal(0,4.37,0.005) crown A5,S2,O2
```
This will tell Stange et al. 2018's custom script that we are constraining the root of the tree to be 4.37 mya with 0 offset and a deviation of 0.005. For details of what all that means, please see [Stange et al. 2018](https://doi.org/10.1093/sysbio/syy006) and the [snapp prep page](https://github.com/mmatschiner/snapp_prep). With these inputs, we can run snapp_prep.rb to build the xml file.
```bash
ruby snapp_prep.rb -v SNAPP.v5.snp.filtered.biallelic.no.missing.5kthinned.individual.filtered.recode.vcf -t samples.txt -c constraints.txt -l 5000000 -x snapp_run.xml
```
The script is run with `ruby`. The `-l` flag tells the script how many generations we want Beast2 to un for, and the `-x-` flag denotes the output name.

## Running Beast2
In full disclosure, I (William Davis) have not been able to get Beast2 to completion on the 'Omics server. Instead, I ended up using the [CIPRES server](http://www.phylo.org/), which, as of 20220317, is free for U.S. citizens. If you wish to try beast on the server, then simply run `beast -threads -2 -instances 2 -working snapp_run.xml`.

## Visualizing Beast trees

First, we use [DensiTree](https://www.cs.auckland.ac.nz/~remco/DensiTree/) to view the trees on our local machine.

Next we import the trees into [TreeAnnotator](https://beast.community/treeannotator) to estimate support and to claculate the node ages.

Change the following settings:
*Burnin percentage: 10
*Target tree type: maximum clade credibility tree*Node heights: mean hieghts
*Input tree file: snapp.trees
*Output file: snapp.tre

Then we viewed the `snapp.tre` file in [FigTree](https://github.com/rambaut/figtree/releases).

Change the following settings:
*Check "Node labels"
	*"Node labels" -> "Display" -> "posterior"
*Check "Scale axis" -> uncheck "Show grid" -> check "Reverse axis"
*"Time scale" -> "Scale root to" -> "Root age"

From the above, we can get a nice enough tree and view the results. However, we can build a nicer tree using ggplot2 in R by following the online ggtree book, specifically sections [3.3.2.1](https://guangchuangyu.github.io/ggtree-book/chapter-treeio.html#functions-demonstrated), [4.3.5](https://guangchuangyu.github.io/ggtree-book/chapter-ggtree.html#tree-annotation-using-data-from-evolutionary-analysis-software), and [4.3.3](https://guangchuangyu.github.io/ggtree-book/chapter-ggtree.html#layer). We can also use the [deeptime](https://github.com/willgearty/deeptime) package to add scales and names. Below is an example script. The specifics have been omitted, but the below example is based on the partridge project. Thus, the dates/ages and names will need to be adjusted based on your data.  
```R
#Time divergence Beast tree

#Load libraries
library(tidyverse)
library(readxl)
library(ggtree)
library(ggtreeExtra)
library(deeptime)
library(ggfittext)
library(phytools)
library(scales)
library(RColorBrewer)
library(patchwork)
library(ggstance)
library(treeio)
library(ape)

#Set working directory
setwd("/PATH/")

#set resolution
ppi <- 1000

#Load data

tip_labels <- read.csv("sample_info.csv", header=TRUE)

snapp_tree <-read.beast("snapp.tre")

Tree_plot <- ggtree(snapp_tree) %<+% tip_labels +
geom_tiplab(aes(label = label_pretty)) +
geom_range("height_0.95_HPD", color="#08306b", size=4, alpha=0.90) +
geom_text2(aes(label=round(as.numeric(height), 4), 
                x=branch), vjust=1.5) +
geom_text2(aes(label=round(as.numeric(posterior), 2), 
                 subset=as.numeric(posterior)> 0, 
                 x=branch), vjust=-0.5) +
xlim(NA, 5)

tiff("Time_divergence_tree_beast.tiff",
width= 9.4*ppi,
height=7.4*ppi,
res = ppi,
compression= "lzw")
print(Tree_plot)
dev.off()


Tree_plot_w_ages <-ggtree(snapp_tree) %<+% tip_labels +
			geom_tiplab(aes(label = label_pretty)) +
			geom_range("height_0.95_HPD",
					color="#08306b",
					size=4,
					alpha=0.90) +
			geom_text2(aes(label=round(as.numeric(height), 4), 
                			x=branch), vjust=1.5) +
			geom_text2(aes(label=round(as.numeric(posterior), 2), 
                 			subset=as.numeric(posterior)> 0, 
                 			x=branch), vjust=-0.5) +
			coord_geo(dat= "epochs",
					pos= "b",
					xlim = c(-5,1.5),
					ylim = c(-0,6.1),
					neg = TRUE,
					abbrv = FALSE,
					size = "auto",
					fittext_args = list(size = 10)) +
			scale_x_continuous(breaks=seq(-5, 0),
			labels=abs(seq(-5, 0))) +
			theme_tree2()

revts(Tree_plot_w_ages)

tiff("Time_divergence_tree_beast_w_ages.tiff",
width= 9.4*ppi,
height=7.4*ppi,
res = ppi,
compression= "lzw")
print(revts(Tree_plot_w_ages))
dev.off()
```
The `sample_info.csv` is formatted as follows:
```bash
Sequence_ID,label_pretty
A5,August species name individual 5
S2,September species name individual 2
O2,October species name individual 2
```

Congrats! Woot! Now you have a time divergence tree!
