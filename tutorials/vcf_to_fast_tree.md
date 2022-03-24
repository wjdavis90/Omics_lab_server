Once we have our windows of interest, it is always a good idea to build gene trees of them to see if the inferred evolutionary history matches with what one would predict based on the Fst values (or other estimates of divergence). We can do this very quickly by building fasta files of the SNPs and using [fasttree](http://www.microbesonline.org/fasttree/), which has decent online documentation. Depending on the number of windows one has, this can be time consuming and is a good opportunity to make use of [parallel](https://www.gnu.org/software/parallel/sphinx.html).

First, we need will need a list of the windows. Most of the time, we want such a list to be in [bed format](https://en.wikipedia.org/wiki/BED_(file_format)) such that column 1 is the chromosome/scaffold, column 2 is the start of the window, and column 3 is the end of the window. **Note:** most of the time bed files do not have headers. E.g., `exciting.window.bed`:
```bash
1 1000000 1001500
2 1000000 1001500
3 1000000 1001500
4 1000000 1001500
5 1000000 1001500
```
**Note:** Columns are separated by tabs.

Next, we need to write a VCF for each window of interest. With only a few windows, this can easily be done by hand. However, with the partrige project, we had 56 windows. I used a combination of `ctrl + H` and regular expression *magic* in [Notepad++](https://notepad-plus-plus.org/) in combination with the `paste` unix command to create a `.sh` file with all of the commands for each window. I apologize for that shoddy/black box explanation that ultimately tells you nothing. But this is a good example of where I did not properly record what I did and so cannot easily repeat it. One could probably use `parallel` by providing more than one input variable. So, if one figures out a better way of doing this using `parallel` or something else, please update this section. Anyway, the `.sh` file will have a series of commands like the following:
```bash
#!/bin/sh

vcftools --vcf final.v1.snp.filtered.Q100.GQ10.vcf.recode.vcf --recode --out final.v1.snp.filtered.Q100.GQ10.scaff.1.1000000.1001500 --chr 1 --from-bp 1000000 --to-bp 1001500 &&

echo "Vcf files written."
```
I used the `&&` to tell the shell only to run the next command if and only if this only finished without error. One uses the `sh` command to run the script.

Now, we can use `parallel` and the python script `vcf2phylip.py` to create a fasta file of the SNPs in each window. Before we do, we need to list the windows as single entry per line, e.g., `exciting.window.list`:
```bash
1.1000000.1001500
2.1000000.1001500
3.1000000.1001500
4.1000000.1001500
5.1000000.1001500
```
We use the transformed list as the input into `parallel`:
```bash
cat exciting.window.list | parallel -j 8 vcf2phylip.py -i final\.v1\.snp\.filtered\.Q100\.GQ10\.scaff\.{1}\.recode\.vcf --phylip-disable -f --output-folder \/PATH\/fasttree\/ --output-prefix {1}
```
For an explanation of the extra `\` see [Example 3](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/parallel_examples.md#example-3).

Now we can use FastTree to infer gene trees. First, we need to set an environmental variable `export OMP_NUM_THREADS=1`. Then we can make use of `parallel` again.
```bash
cat exciting.window.list | parallel -j 8 FastTreeMP -nt -gtr -fastest {1}\.min4\.fasta \> {1}\.tre 2\> {1}\.log
```

We can visualize the trees in R. I will only do one tree as an example. To start, we need an information file in order to make the tip labels pretty. E.g., `population_info.csv`:
```bash
Sequence_ID,label_pretty
A1,August
A2,August
A3,August
A4,August
A5,August
S1,September
S2,September
S3,September
S4,September
S5,September
O1,October
O2,October
O3,October
O4,October
O5,October
```
One can add additional columns that can be used to color the tips and other things if one wants.


