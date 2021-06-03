# Master list of programs

**Purpose:** This document is a list all programs/tools on the server and the PATH to them. It is also annotated to include special directions for running a program if need be.

**Tip:** If one is going to be using one of these programs regularly, it is recommended one edits one's `.bash_profile` file in one's home directory to export it to PATH every time one logs in. Example using Dsuite:

This is what `.bash_profile` looks like before editing.

```bash
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
```

Using vi or vim, one adds `PATH=$PATH:/scratch/Tools/Dsuite/Build/` and `PATH=$PATH:/scratch/Tools/Dsuite/utils/` to it so that it looks like this.

```bash
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin
PATH=$PATH:/scratch/Tools/Dsuite/Build/
PATH=$PATH:/scratch/Tools/Dsuite/utils/

export PATH
```

NOTE: One may need to set PATH to a subdirectory, such as `/bin/`, `/Build/`, `/utils/`, or `/src/`!

Below I list the programs grouped by their path.

## /home/sangeet/UserDirectories/Sangeet/Tools/

- admixture_linux-1.3.0
- AGAT
- App-cpanminus-1.5017
- bedtools
- bin
- bowtie2-2.4.1-linux-x86_64
- busco_downloads
- cgal-0.9.6-beta
- cmake-3.17.1-Linux-x86_64
- cmake-3.17.1-Linux-x86_64.sh
- core-rnammer
- fsc26_linux64
- gatk-4.1.3.0
- genometools-1.5.9
- hapflk1.1_linux64
- hapflk-1.4
- HMMER
- hmmer-3.3.2
- hyphy-2.5.11
- include
- install.sh
- javase-jdk13-downloads.html
- jellyfish-2.3.0
- kallisto
- kraken2-2.0.8-beta
- maker
- maker-functional
- MCScanX
- Miniconda
- myconfig.ini
- ncbi-blast-2.10.0+
- obsutil_linux_amd64_5.1.13
- OrthoFinder_2.3
- OrthoFinder-2.5.2
- PGDSpider_2.1.1.5
- plink-1.07-x86_64
- Reapr_1.0.18
- RepeatMasker
- RERconverge-0.3.0
- RNAMMER
- salmon-latest_linux_x86_64
- samtools-1.10
- satsuma2
- satsuma-code-0
- signalp-5.0b
- Spines_release_1.15
- sqlite-tools-linux-x86-3340100
- supernova-2.1.1
- tguenther-bayenv2_public-2b2b7f20bb62
- tmhmm-2.0c
- TransDecoder-TransDecoder-v5.5.0
- trinityrnaseq-v2.9.1
- v0.3.0.tar.gz
- vcftools_0.1.13

# /scratch/Tools
- anaconda3
  - Please see the [instructions](https://github.com/wjdavis90/Omics_lab_server) on how to activate and initialize conda before use.
- angsd
- beast
- cutadapt-3.4
- Dsuite
	- Must do `export LD_LIBRARY_PATH=/scratch/Tools/local/lib64/` before trying to run.
	- Program packages are in `/Build/` and plotting is in `/utils/`.
  - The instructions can be found [here](https://github.com/millanek/Dsuite) and a there is a [tutorial](https://github.com/millanek/tutorials/tree/master/analysis_of_introgression_with_snp_data)
- gsl-2.6
- htslib
- parallel-20210522
- snapp_prep-master
- QuIBL
- GCC 4.9.4
	- The executables are in `/scratch/Tools/local/bin`
- EMBOSS
	- The executables are in `/scratch/Tools/local/bin`
- treemix-1.13
- TrimGalore-0.6.6

# /scratch/Tools/scripts
- analyze_tree_asymmetry.rb
- blast_taxonomy_report.pl
- construct_unaln_files_180325.pl
- extract_blocks.rb
- fastaToTab
- fill_seq.rb
- getClassFasta.pl
- get_fixed_site_gts.rb
- get_number_of_pi_sites.rb
- get_number_of_variable_sites.rb
- get_parsimony_score.sh
- gff3ToFasta
- mask_imputed_gts.rb
- plot_d.rb
- plot_f4ratio.rb
- plot_fixed_site_gts.rb
- plot_ggi.rb
- plotInvestigateResults.R
- plot_tree_asymmetry.rb
- print_tetramer_freqs_deg_filter_esom_VD.pl
- remove_seqs_if_all_gap.py
- summarize_ggi.sh
- tabToFasta
- vcf2phylip.py

# Misc.
- /opt/microsoft/omsagent/ruby/bin/ruby

