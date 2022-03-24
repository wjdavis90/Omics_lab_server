 [Treemix](https://bitbucket.org/nygcresearch/treemix/wiki/Home) is one (of many) ways too look at population structure and to infer the history of those populations. The paper is available [here](https://doi.org/10.1371/journal.pgen.1002967), and the pdf manual, which can be downloaded [here](https://bitbucket.org/nygcresearch/treemix/downloads/) is pretty good. By far, the hardest part is converting the VCF file into something treemix will read. 
 
 The first step to have a test file with a list of the individuals and the population they belong to. For this example, it will be named `population_info.txt`. One population **must** be assigned to the outgroup. For this example, I will use the August population as the outgroup.
 ```bash
 A1 Outgroup
 A2 Outgroup
 A3 Outgroup
 A4 Outgroup
 A5 Outgroup
 S1 September
 S2 September
 S3 September
 S4 September
 S5 September
 O1 October
 O2 October
 O3 October
 O4 October
 O5 October
 ```
 
 Next we will use `populations` within [Stacks](https://catchenlab.life.illinois.edu/stacks/comp/populations.php) to convert the VCF file into a tree mix input file. Another option is to use the [Popgen Pipeline Platform](https://ppp.readthedocs.io/en/latest/PPP_pages/intro.html), which is available on the server within the [conda environment](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/using_conda.md#conda-environments-available-on-the-server) py-popgen.
 
 **Note:** In order to run Stacks and Treemix on the server, one must set the LD_LIBRARY_PATH [environmental variable](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/) so that these programs look for the approriate libraries in the local installation instead of `/usr/local/`, which is the default. One does this using the `export` command as shown below. As well, `populations` requires the full path to where you want the output, which is designated with the `-O` flag.
 ```bash
 export LD_LIBRARY_PATH=/scratch/Tools/local/lib/:/scratch/Tools/local/lib64/

populations -V final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf -M population_info.txt -O /PATH/Treemix --treemix
```

It is vital to open the output at this step and remove the headers that `populations` added. Otherwise Treemix will give the following error:
```bash
TreeMix v. 1.13
$Revision: 231 $

ERROR: Line 0 has 3 entries. Header has 8
```

Once, the header has been removed, gzip the `populations` output and run Treemix:
```
gzip final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf.p.treemix

treemix -i final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf.p.treemix.gz -root Outgroup -k 1000 -o Populations
```

This should produce 7 files all with the name that you provide with the `-o` flag. These can be used to build a graph using R using a script called `plotting funcs.R` that comes as part of the Treemix source code. On the server, it is located here `/scratch/Tools/treemix-1.13/src`. Below is an example of how to use this script to produce the graph.
```R
#Plotting TreeMix results

#set working directory
setwd("/PATH/")

#Set ppi
ppi <-1000


#load the plotting funtion
source("/scratch/Tools/treemix-1.13/src/plotting_funcs.R")

#Plot the trees
plot_tree("Populations")

#Save plot
tiff("Populations_treemix.tiff",
width= 7.4*ppi,
height=4.7*ppi,
res = 1000,
compression= "lzw")
plot_tree("Populations")
dev.off()
```
