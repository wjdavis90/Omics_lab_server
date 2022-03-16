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

