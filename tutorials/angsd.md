Angsd does have a [home page/manual](http://www.popgen.dk/angsd/index.php/ANGSD), but I found it super frusterating. First, it's still in beta, and there are sections that remain to be written. Second, no where does it list the various options/flags and what they mean. Third, explanations are often vague and sometimes are missing crucial details. Thus, I wrote this to document some of the problems I encountered. 

Angsd can be used for many things, which one can view on its home page. Our lab primarily uses it to calculate [population branch statistics](http://www.popgen.dk/angsd/index.php/Fst), which were developed by [Yi et al. 2010](https://doi.org/10.1126/science.1190371), and the [Wu-Watterson Estimator/theta](https://en.wikipedia.org/wiki/Watterson_estimator)[^1] to estimate the [effective population size](https://en.wikipedia.org/wiki/Effective_population_size). Since they require the same first steps, I will treat them together.

Before we begin, make sure you add `/scratch/Tools/angsd` AND `/scratch/Tools/angsd/misc` to your path, either by running `export PATH=$PATH:/scratch/Tools/angsd` AND `export PATH=$PATH:/scratch/Tools/angsd/misc` or by [adding them to your bash profile](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/setting_PATH.md).

**NOTE:** As of this writing (20220311), angsd cannot properly handle multi-threading, as revealed in this [post](https://github.com/ANGSD/angsd/issues/376). Everything must be run on one thread!

First, we need to estimate [site frequency spectrums (SFS)](http://www.popgen.dk/angsd/index.php/RealSFSmethod) for each population. Angsd has a fairly detailed explanation of how to do so [here](http://www.popgen.dk/angsd/index.php/SFS_Estimation). However, the explanation is primarily built around the use of bam files, but sometimes we need to start with a [vcf file](https://samtools.github.io/hts-specs/VCFv4.2.pdf). The manual is not so good at explaining how to do this. It merely says `angsd -vcf-gl`. However, this does not work if your vcf file was made with vcftools! If that is the case, then one has to use `-vcf-pl` instead of `-vcf-gl`, which is listed no where in the manual, and I only found it because luckily [someone else asked the angsd community directly](https://giters.com/ANGSD/angsd/issues/383). As well, one needs to have a genotype likelihood file, which angsd cannot create from a vcf. Thus, use bam files whenever possible. When starting from a vcf file, use 
```bash
angsd -vcf-pl your.file.vcf -doSaf 1 -doMajorMinor 1 -anc your.reference.genome.fasta -beagle your.genolike.beagle.gz -P 4 -out your.output.name
realSFS your.output.name.saf.idx > your.output.name.sfs
```

After one has all of the SFS estimated, one can then compute population branch statistics. To start, we make 1 to 1 comparisons of the populations. For the partridges, we used `-nSites 100000000` in order to speed up the computational times/reduce memory load. This can be adjusted as necessary.
```bash
realSFS population1.saf.idx population2.saf.idx -nSites 100000000 > 2dsfs.population1.population2
realSFS population1.saf.idx popualtion3.saf.idx -nSites 100000000 > 2dsfs.population1.population3
realSFS population2.saf.idx population3.saf.idx -nSites 100000000 > 2dsfs.population2.population3
```
Then we can do all three. It turns out that order is critical! (Not that the manual says that.) Thus, pay attention to what order you list the populations and the comparisons. (And you may have to change it until it works!)
```bash
realSFS fst index population1.saf.idx population2.saf.idx population3.saf.idx -sfs 2dsfs.population1.population2 -sfs 2dsfs.population1.population3 -sfs 2dsfs.population2.population3 -fstout your.output.name.pbs -whichFST 1
```
After that is complete, we can get population branch statistic values on a per window basis. The standard window size is 15,000.
```bash
realSFS fst your.output.name.fst.idx -win 15000 -step 15000 -whichFST 1 > final.output.name
```





[^1]: As noted in the linked Wikipedia article, [Margaret Wu](https://en.wikipedia.org/wiki/Margaret_Wu) contributed to the development of the Watterson estimator/theta (among many other things!) but was not included as an author on the paper, merely an acknowledgment. To me (William J. Davis) this is an injustice; therefore, I (William J. Davis) will forever refer to it as the Wu-Watterson estimator/theta, and I sincerely hope others will adopt this as well.
