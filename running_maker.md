In order to run maker, please add the following to your `.bash_profile` file. Instructions on how to do so can be found [here](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/setting_PATH.md).

```
PATH=$PATH:/home/sangeet/UserDirectories/Sangeet/Tools/RepeatMasker/
PATH=$PATH:/home/sangeet/anaconda3/envs/py3/bin/
PATH=$PATH:/scratch/Tools/maker/src/bin/

export PATH

LD_PRELOAD=$LD_PRELOAD:/home/sangeet/anaconda3/lib/libmpi.so

export LD_PRELOAD
```

You may also have to add the following to your `.bashrc` file.

```
PATH="/home/wdavis34/perl5/bin${PATH:+:${PATH}}"; export PATH;
PERL5LIB="/home/wdavis34/perl5/lib/perl5${PERL5LIB:+:${PERL5LIB}}"; export PERL5LIB;
PERL_LOCAL_LIB_ROOT="/home/wdavis34/perl5${PERL_LOCAL_LIB_ROOT:+:${PERL_LOCAL_LIB_ROOT}}"; export PERL_LOCAL_LIB_ROOT;
PERL_MB_OPT="--install_base \"/home/wdavis34/perl5\""; export PERL_MB_OPT;
PERL_MM_OPT="INSTALL_BASE=/home/wdavis34/perl5"; export PERL_MM_OPT;
```
