Add this to your `.bash_profile`
```
PATH=$PATH:/home/sangeet/UserDirectories/Sangeet/Tools/ncbi-blast-2.10.0+/bin/
PATH=$PATH:/home/sangeet/UserDirectories/Sangeet/Tools/RepeatMasker/
PATH=$PATH:/scratch/Tools/mpich-install/bin/
PATH=$PATH:/scratch/Tools/exonerate-2.2.0-x86_64/bin/
PATH=$PATH:/scratch/Tools/maker/bin/

export PATH
```

Then can call `mpiexec -n 4 maker -h`.

May need to add this to your `.bashrc`.
```
PATH="/home/wdavis34/perl5/bin${PATH:+:${PATH}}"; export PATH;
PERL5LIB="/home/wdavis34/perl5/lib/perl5${PERL5LIB:+:${PERL5LIB}}"; export PERL5LIB;
PERL_LOCAL_LIB_ROOT="/home/wdavis34/perl5${PERL_LOCAL_LIB_ROOT:+:${PERL_LOCAL_LIB_ROOT}}"; export PERL_LOCAL_LIB_ROOT;
PERL_MB_OPT="--install_base \"/home/wdavis34/perl5\""; export PERL_MB_OPT;
PERL_MM_OPT="INSTALL_BASE=/home/wdavis34/perl5"; export PERL_MM_OPT;
```
