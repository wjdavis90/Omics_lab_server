# Welcome!
Welcome to the [OMICS lab](https://theomicslab.wordpress.com/) at Kent State University run by Dr. Sangeet Lamichhaney! 

This GitHub repository contains useful information about the lab server as well as various analyses frequently run in the lab. In this repository, you will find:
* [A list of programs available on the server](https://github.com/wjdavis90/Omics_lab_server/blob/main/program_list.md)
* [Tutorials](https://github.com/wjdavis90/Omics_lab_server/tree/main/tutorials) on how to use those programs
* Tutorials and data for the [October 2021 Bioinformatics Boot camp](https://github.com/wjdavis90/Omics_lab_server/tree/main/tutorials/2021_Bio_Info_Bootcamp)
* Other useful tips/hints

A common pipeline/work flow for the 'Omics lab is presented below. For all the examples, I assume three populations of 5 individuals each. I named the populations "August", "September", and "October", and the individuals are labelled as the first letter of the month (i.e., population name) followed by a number, e.g., A1.

1. [Align reads to a reference genome and call SNPS](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/Aligning_reads_calling_SNPS.md)
2. [Calculate intra-population statistics](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/intra_population_stats.md)
3. [Determine relatedness of individuals within populations](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/relatedness.md)
4. [Use Treemix to look at migration among populations](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/treemix.md)
5. Estimate divergence among populations using either [Fst](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/fst.md) or [Population branch statistics](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/angsd.md)
6. Identify windows of interest
7. [Build phylogenetic trees of the windows of interest]()
8. [Run SNPeff]()
