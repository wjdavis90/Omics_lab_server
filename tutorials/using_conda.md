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

- [VAMB](https://github.com/RasmussenLab/vamb)
- [admixture](https://dalexander.github.io/admixture/index.html)
- [agat](https://github.com/NBISweden/AGAT)
- [bcftools](https://samtools.github.io/bcftools/bcftools.html)
- [busco](https://busco.ezlab.org/)
- [cactus](https://github.com/ComparativeGenomicsToolkit/cactus)
  -May not work/don't use 
- catadaptenv
- [haltools](https://github.com/ComparativeGenomicsToolkit/hal)_
- [hapflk](https://github.com/bcm-uga/SSMPG2017/blob/master/Presentations/hapflk/hapflk.org)
- [py-popgen](https://pypi.org/project/py-popgen/)
- r_env
- [raxml](https://github.com/stamatak/standard-RAxML)
- rsem
- star
- [structure](https://web.stanford.edu/group/pritchardlab/structure.html)
  - includes [fastStructure](https://github.com/rajanil/fastStructure)
  - [structureHarvester](http://alumni.soe.ucsc.edu/~dearl/software/structureHarvester/)
