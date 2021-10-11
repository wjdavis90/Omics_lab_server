# How to download/install within an unix/unix-like system

If you are fortunate, then whatever program you need will come as pre-compiled binaries and all you have to do is download, decompress, and palce in the folder that you want them. Or, the program can be downloaded and installed within a [conda environment](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/using_conda.md). A little more complicated is if you have to compile the program yourself. Usually, that is not too complicated, and all you have to do is follow the instructions in the manual. If you ever need to do this on the server, come speak with me as there are some server specific things that may have to be done.

For boot camp, we are going to download blast, which comes has pre-compiled packages. First, navigate into the /Tools/ subfolder (assuming that you are in the 2021-BioInfo_Boot_Camp folder).
```bash
#For linux
wget https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-2.12.0+-x64-linux.tar.gz

#for macs
wget https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-2.12.0+-x64-macosx.tar.gz

#Then we decompress
tar -xfv ncbi-blast-2.12.0+-x64-linux.tar.gz
tar -xfv ncbi-blast-2.12.0+-x64-macosx.tar.gz
```

There! The program is downloaded and ready to use. Now, to avoid having to put in the full path every time we want to call a blast program, we can [set the path to the bin folder to PATH by editing the bash profile](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/setting_PATH.md).
