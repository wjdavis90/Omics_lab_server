# Getting to Know R

**Learning objectives**

6.	how to install packages and load libraries in R
7.	how to import data into R
8.	[how to manipulate data in R](https://www.tidyverse.org/)
9.	[how to use ggplot2](https://r-graphics.org/)

[R](https://www.r-project.org/about.html) is a programming language and environment used for statistics and graphics. It is based on S, which is a language and environment that, like Unix, was developed by programmers at AT&T. 

The most striaghtforward way to install a packacge is `install.packages("package")` with the word package replaced with the name of the package, e.g. `install.packages("tidyverse")`. Sometimes it will be necessary to install a package from GitHUb, in which case we would use `devtools::install_github("thomasp85/patchwork")`. If the package is on [Bioconductor](https://www.bioconductor.org/install/), then we will need to use `BiocManager::install()`.

After a package is installed, we can load it with `library()`, e.g., `library(tidyverse)`.

There are many ways to import dat into R. The ones will will be using are `read.csv()` and `read.table()`.
