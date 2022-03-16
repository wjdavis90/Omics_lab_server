GNU parallel usefule tool that can be used in place of wait scripts and/or for loops. It has a decent [tutorial](https://www.gnu.org/software/parallel/parallel_tutorial.html) and a series of youtube [videos](https://www.youtube.com/playlist?list=PL284C9FF2488BC6D1). I ahve not read too much of the tutorial or watched the videos. Most of what I know comes from examples of other people using it, which is why I am going to document the ways I have used it here.

**Note:** If you use parallel, you **MUST** cite it in your publication. As of 20220316, the citation is:
```bash
Tange, O. (2021, May 22). GNU Parallel 20210522 ('Gaza'). Zenodo. https://doi.org/10.5281/zenodo.4781603
```

## Example 1

I wanted to run a [hapflk analysis](https://github.com/bcm-uga/SSMPG2017/blob/master/Presentations/hapflk/hapflk.org) on a genome, but it has to be run on a per chromosome or, in our case, a per scaffold basis. This means creating an input file for each scaffold. When one has thousands of scaffolds, to do so manually is tedious and time consuming. To do so with a for loop is just as time consuming if slightly less tedious. This is where parallel can be used! First, I created a list of the scaffolds and pragmatically named it `scaffolds`. Then I ran the line below.
```bash
cat scaffolds | parallel -j 4 vcftools --vcf example.input.vcf --recode --out example.output.scaffold.{1} --chr {1}
```
What happens is that `cat scaffolds` begins reading `scaffolds` to STOUT line by line. The `|` redirects that output into parallel. Paralell takes those names and encodes them into a variable designated (in my example) by `{1}`. It then begins a series of jobs, each on a separate core/CPU/processor. The number of jobs to start at a time is designated by the `-j` flag. I tend to run 4 at a time. The job called is `vcftools --vcf example.input.vcf --recode --out example.output.scaffold.{1} --chr {1}`, which is to make a new vcf file containing only a particular scaffold (chromosome). Altogether, parallel will keep running vcftools in batches of 4 until it reaches the end of the list of scaffolds. Much faster than doing them one at a time. 

## Example 2

For another usage, I wanted to pull all of the unmapped reads out of 30 bam files, create fastq files of the unmapped reads, and then do a quick assembly of those reads. Fist, I created a list of the bam files and named it `bam_list`. The commands I ran were:
```bash
cat bam_list | parallel -j 5 samtools view -b -f 4 {1}.transcript.bam \> unmapped.{1}.bam

cat bam_list | parallel -j 5 bam2fastq -o {1}_unampped_reads#.fastq unmapped.{1}.bam

cat bam_list | parallel -j 2 Trinity --seqType fq --max_memory 50G  --left {1}_unampped_reads_1.fastq --right {1}_unampped_reads_2.fastq --CPU 2 --output trinity_{1}
```
In each, `cat bam_list` reads the list to STOUT and `|` redirects it to parallel, which encodes the names within the list into a variable designated by `{1}`. For the first line, parallel runs `samtools view -b -f 4 {1}.transcript.bam \> unmapped.{1}.bam` in groups of 5 (designated by the `-j` flag), which creates a new bam file containing only the unmapped reads. In the second line, parallel runs `bam2fastq -o {1}_unampped_reads#.fastq unmapped.{1}.bam`, which creates three fastq files containing the unmapped reads. The last line has parallel running an assembler to make assemblies out of the unmapped reads. 

**Note:** I struggled for a while to get the first line to run properly (i.e., produce the output I wanted). I kept producing a single bam file called `unmapped.{1}.bam` instead of a new bam file for each of the input bam files. After a while, i figured out that I had to [escape](https://linux.die.net/abs-guide/escapingsection.html) the ">". In other words, I had to indicate that the ">" was part of the command I wanted parallel to run and not where I was telling parallel to redirect the output. Written as `cat bam_list | parallel -j 5 samtools view -b -f 4 {1}.transcript.bam > unmapped.{1}.bam`, parallel was running `samtools view -b -f 4 {1}.transcript.bam` and putting all of the output into a single file `unmapped.{1}.bam`. 

## Example 3
For the last example, I wanted to use `awk` to make count data for 10 patterns across 406 files. Really, really, really did not want to do it by hand or one at a time. Thus, I wrote the following script using parallel to make the counts.
```bash
#!/bin/sh
cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 0 \&\& \$2 \=\= 2\) \{print \$0\}\}\' {1} \| wc -l \> {1}_02_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 2 \&\& \$2 \=\= 0\) \{print \$0\}\}\' {1} \| wc -l \> {1}_20_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 0 \&\& \$2 \=\= 0\) \{print \$0\}\}\' {1} \| wc -l \> {1}_00_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 2 \&\& \$2 \=\= 2\) \{print \$0\}\}\' {1} \| wc -l \> {1}_22_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 0 \&\& \$2 \=\= 1\) \{print \$0\}\}\' {1} \| wc -l \> {1}_01_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 1 \&\& \$2 \=\= 0\) \{print \$0\}\}\' {1} \| wc -l \> {1}_10_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 1 \&\& \$2 \=\= 1\) \{print \$0\}\}\' {1} \| wc -l \> {1}_11_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 2 \&\& \$2 \=\= 1\) \{print \$0\}\}\' {1} \| wc -l \> {1}_21_count &&

cat comparison_list | parallel -j 4 awk \'\{if \(\$1 \=\= 1 \&\& \$2 \=\= 2\) \{print \$0\}\}\' {1} \| wc -l \> {1}_12_count &&

cat comparison_list | parallel -j 4 wc -l {1} \> {1}_total_SNP
```
I won't bother explaining in depth as I did for the examples above. The gist is that I ran 4 instances of `awk` counting each of the 10 patterns in each of the 406 files. However, it is important to explain one thing. If I only wanted to run the count for one file, it would look like this `awk '{if ($1 == 0 && $2 == 2) {print $0}}' example_file | wc -l > example_output` but to run it within parallel it looked like this `awk \'\{if \(\$1 \=\= 0 \&\& \$2 \=\= 2\) \{print \$0\}\}\' {1} \| wc -l \> {1}_02_count`. I had to use "\" to escape out all of the non-alphanumeric characters so that parallel would know they were part of the `awk` command and not part of parllel. **The morale:** Sometimes you will have to write something that will look weird/very different than what you would write to run on a single file in order for it parallel to read it properly!
