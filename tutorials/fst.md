[Fst](https://en.wikipedia.org/wiki/Fixation_index) is, in general, a measure of how divergent two popualtions are. It is often used as a way to look for regions of the genome under selection since strong selective forces should push the values toward the extremes. Fst ranges from 0 to 1, with values closer to 0 indicating that populations are very similar and values closer to 1 indicating populations are very divergent/dissimilar. **Note:** Because of the way computer programs dot he math to estimate Fst, we very often get values below 0. This is an error.

First, we will need to generate text files listing the individuals that belong to each population. We can use the same text files that we used to [estimate intra population statistics](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/intra_population_stats.md). Thus, for this example, I will reuse `august.individuals`.
```bash
A1
A2
A3
A4
A5
```

From there, we can tell vcftools to calculate Fst between populations. Typically, we do this in non-overlapping windows of 15000 base pairs.
```bash
vcftools --vcf 3final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --out august.september --weir-fst-pop august --weir-fst-pop september --fst-window-size 15000 --fst-window-step 15000

vcftools --vcf 3final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --out august.october --weir-fst-pop august --weir-fst-pop october --fst-window-size 15000 --fst-window-step 15000

vcftools --vcf 3final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --out september.october --weir-fst-pop october --weir-fst-pop september --fst-window-size 15000 --fst-window-step 15000
```

Now comes the hard part. What we do is want to do is visualize the results to see how Fst varies across the genome and then devise a series of cutoffs to select regions of interest. If one is looking only at two populations, one can simply plot the raw values. However, if one is comparing more than two populations, it helps to standardize the values such that they are comparable. The most common way to do this is to use what is known in statstics as the [standard score or the z-score](https://en.wikipedia.org/wiki/Standard_score). The z-score is the number of standard deviations a data point is above or below the mean for the statistic, in this case, Fst. For a given data point, the z-score is the value minus the mean divided by the standard deviation. (See the linked [Wikipedia article](https://en.wikipedia.org/wiki/Standard_score#Calculation) for the formula in mathematical notation.) From here on out, I will refer to the z-score Fst as the ZFst. We will filter the data a bit to remove contigs less than 15000 base pairs (the window size) and with fewer than a certain number of SNPs/variants. (A general rule of thumb is 20.) We then will generate a [Manhattan](https://en.wikipedia.org/wiki/Manhattan_plot) of the Zfst values. I do all of this in R, but the z-score calculation and filtering can also be done (usually much faster) using `awk`.

**Note:** Catalina has a much better script for generating Manhattan plots and correctly ordering the scaffolds along the x-axis. Thus, I am only going to show examples of how to calculate Zfst and filter the data and make joy plots with the raw Fsts.

```R
#Fst

#load Libraries
library(tidyverse)
library(patchwork)
library(RColorBrewer)
library(ggridges)


#set output resolution
ppi <- 1000


#Set working directory
setwd("/PATH/")


#Load fst & transform data

august.september.fst1 <-read.table("august.september.windowed.weir.fst",
				as.is=T,
				header=T)%>%
		mutate(august_september_Zfst = (MEAN_FST - mean(MEAN_FST))/sd(MEAN_FST)) %>%
		rename(august_september_N_VARIANTS = N_VARIANTS) %>%
		rename(august_september_WEIGHTED_FST = WEIGHTED_FST) %>%
		rename(august_september_MEAN_FST = MEAN_FST)

august.october.fst1 <-read.table("august.october.windowed.weir.fst",
				as.is=T,
				header=T)%>%
		mutate(august_october_Zfst = (MEAN_FST - mean(MEAN_FST))/sd(MEAN_FST)) %>%
		rename(august_october_N_VARIANTS = N_VARIANTS) %>%
		rename(august_october_WEIGHTED_FST = WEIGHTED_FST) %>%
		rename(august_october_MEAN_FST = MEAN_FST)

september.october.fst1 <-read.table("Perdix_Ph_Pd.fst.windowed.weir.fst",
				as.is=T,
				header=T)%>%
		mutate(september_october_Zfst = (MEAN_FST - mean(MEAN_FST))/sd(MEAN_FST)) %>%
		rename(september_october_N_VARIANTS = N_VARIANTS) %>%
		rename(september_october_WEIGHTED_FST = WEIGHTED_FST) %>%
		rename(september_october_MEAN_FST = MEAN_FST)


#Filter the Zfst tables
#Create vectors that will filter by the number of times a chromosome appears as a proxy for size
#Then use the vector in subset! (Catalina is an amazing person for explaining how to do this to me!)

august_september_window_count <- august.september.fst1 %>%
			count(CHROM) %>%
			arrange(desc(n))

august_september_window_sub <-subset(august_september_window_count, n>=3)

august_september_vector <- as.vector(august_september_window_sub[['CHROM']])

august.september.fst2 <- subset(august.september.fst1, CHROM %in% august_september_vector)

august_october_window_count <- august.october.fst1 %>%
			count(CHROM) %>%
			arrange(desc(n))

august_october_window_sub <-subset(august_october_window_count, n>=3)

august_october_vector <- as.vector(august_october_window_sub[['CHROM']])

august.october.fst2 <- subset(august.october.fst1, CHROM %in% august_october_vector)

september_october_window_count <- september.october.fst1 %>%
			count(CHROM) %>%
			arrange(desc(n))

september_october_window_sub <-subset(september_october_window_count, n>=3)

september_october_vector <- as.vector(september_october_window_sub[['CHROM']])

september.october.fst2 <- subset(september.october.fst1, CHROM %in% september_october_vector)


#filter Zfst tables by number of SNPs

august.september.fst3<- subset(august.september.fst2, august_september_N_VARIANTS >= 20)

august.october.fst3<- subset(august.october.fst2, august_october_N_VARIANTS >= 20)

september.october.fst3<- subset(september.october.fst2, september_october_N_VARIANTS >= 20)


#Now we merge the fst files together

merge1 <- merge(august.september.fst3, august.october.fst3, all.x= TRUE, all.y= TRUE)

merge2 <- merge(merge1, september.october.fst3, all.x= TRUE, all.y= TRUE)

#Retain the columns that we want

Merged_fst_table <- subset(merge2, select = c("CHROM", "BIN_START",
								"BIN_END",
								"august_september_MEAN_FST",
								"august_september_Zfst",
								"august_october_MEAN_FST",
								"august_october_Zfst",
								"september_october_MEAN_FST",
								"september_october_Zfst"))


write.csv(Merged_fst_table, "Zfst_merged_table.csv")

#Talk to Catalina about creating pretty Manhattan plots!

#Pivot the table for making joy plots 
#First we reduce to just the columns we want
#Then we combine "CHROM" and "BIN_START" to make pivoting easier
#We also rename the popualtion comparisons such that we can better control the order that they appear in the joyplot

fst_table_reduced <- Merged_fst_table %>%
				subset(, select=c("CHROM",
							"BIN_START",
							"august_september_MEAN_FST",
							"august_october_MEAN_FST",
							"september_october_MEAN_FST")) %>%
				unite("region", BIN_START:BIN_END, sep= "-") %>%
				unite("window", CHROM:region, sep= ":") %>%
				rename(C = august_october_MEAN_FST) %>%
				rename(B = september_october_MEAN_FST) %>%
				rename(A = august_september_MEAN_FST) %>%
				pivot_longer(!window,
							names_to = "comparison",
							values_to = "MeanFst")

joy_plot_fst <-ggplot(fst_table_reduced,
			aes(x=MeanFst,
				y=comparison,
				fill=comparison)) +
geom_density_ridges(scale=0.9)+
scale_fill_brewer(palette="Paired") +
theme(legend.position="none") +
labs(x="Mean Fst") +
scale_y_discrete(name=NULL,
			breaks=c("A",
					"B",
					"C"),
			labels=c("August & September",
					"September & October",
					"August & October"))


tiff("mean_fst_joy_plot.tiff",
width= 7.4*ppi,
height=9.4*ppi,
res = ppi,
compression= "lzw")
print(joy_plot_fst)
dev.off()
```
