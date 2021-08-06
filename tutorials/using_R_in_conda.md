# Using R within the conda environment

This information comes from the [Anaconda documentation](https://docs.anaconda.com/anaconda/user-guide/tasks/using-r-language/).

First, activate the conda environment: `conda activate r_env`.

From here, you can run R interactively by executing `R` or run a pre written script with `Rscript --vanilla script`.

There are two ways of installing an R package within the environment.

1) `conda install r-packagename` Note, it is important to put "r-" in front of the package name as this tells conda that you want an R package and not an Anaconda package!
2) Within R itself with `install.packages()` or devtools or Bioconductor. 

Try the first method first, as this is the best way to ensure that all of the dependencies, which may not always come from R, are also downloaded and installed.
