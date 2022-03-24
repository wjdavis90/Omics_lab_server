# Introduction

[Conda documentation](https://conda.io/projects/conda/en/latest/user-guide/index.html)

# Using conda to install packages

The code below describes how to initialize the conda in /scratch/Tools so that a user can use it. It can be copied and paste into terminal/console to be run.

```bash
#Activate conda
source /scratch/Tools/anaconda3/bin/activate

#Next, initialize conda
conda init
```

Now, one will have to log out and log back in for the changes to take affect. One will see `(base)` next to the command prompt. This means that you are operating within the base conda environment. If you do not wish to do so, and there are many valid reasons not to do so, simply run `conda deactivate`.

For packages that can be installed with conda, the typical command is `conda install package`. However, it is best to create an environment for a package before installing it. That way the installation does not mess up other packages that may use the same dependencies. For example:

```bash
conda create -n bcftools
conda install -n bdcftools -c bioconda bcftools
```

# conda environments available on the server

To view a list of the conda environments available on the server, use `conda info --envs` or `conda env list`.

**NOTE:** After installing programs into a conda environment, one **MUST** run `chmod -R +777 /scratch/Tools/anaconda3` This ensures that other users can install a conda package after you.

- [admixture](https://dalexander.github.io/admixture/index.html)
- [agat](https://github.com/NBISweden/AGAT)
- [bcftools](https://samtools.github.io/bcftools/bcftools.html)
- [busco](https://busco.ezlab.org/)
- catadaptenv
- haltools
- humann
	- includes [humann3](https://github.com/biobakery/biobakery/wiki/humann3#12-installation) 
- metagenomics
	- includes cutadapt 1.18
	- fastqc
	- hmmer
	- [mafft](https://mafft.cbrc.jp/alignment/software/)
	- megahit
	- metaxa
- [PartitionFinder2](http://www.robertlanfear.com/partitionfinder/)
- [py-popgen](https://pypi.org/project/py-popgen/)
- r_env
	- ['Omics lab tutorial](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/using_R_in_conda.md) 
- rsem
- star
- [structure](https://web.stanford.edu/group/pritchardlab/structure.html)
  - includes [fastStructure](https://github.com/rajanil/fastStructure)
  - includes [structureHarvester](http://alumni.soe.ucsc.edu/~dearl/software/structureHarvester/)
- [raxml](https://github.com/stamatak/standard-RAxML)
