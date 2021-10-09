# Creating and manipulating files

Several types of files exist and, usually, the kind we will be dealing with are [regular files](https://en.wikipedia.org/wiki/Unix_file_types). Regualr files come in several flavors that are usually denoted by their [filename extension](https://en.wikipedia.org/wiki/Filename_extension). This is where one of the key features of unix comes into play: Most of the files we will be working with will be some form of a text file. The simplest is a [plain text file](https://en.wikipedia.org/wiki/Plain_text), but this also includes [vcf files](https://samtools.github.io/hts-specs/VCFv4.2.pdf) and [fasta files](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=BlastHelp). Remember, unix is built around manipulating text files, which means any command that will work on a vcf file will work equally well on a fasta file or any other kind of text file. Thus, unix has near universal compatability and can be applied to many of our bioinformatic needs.

Before we go much further, some vocabulary:
- stout: standard output; where the output of a command is going to go, which is usually the screen
- stin: standard input
- sterr: standard error
- flag: this is usually a '-' followed by a letter (though it can also be a '--' followed by a word. Flags are used to change options within a unix command away from the default. For example, `mkdir ./example/` will make the directory "example" within the current working directory. But, if you want to create both a directory and a subdirectory at the same time, you would use the `-p` flag like so: `mkdir -p ./example/subdirectory/`. 
- | allows us to turn the stout of one command into the stin of another
- \> tells a command to write stout to a file rather than the screen
- \>> tells a command to append stout to an existing file rather than writing a new file or overwriting an existing file
- &> tells a command to write stout and sterr to a file rahter than the screen
- 2> tells a command to write sterr to a log file (used with >)

## Creating files

The simplest way to create a file is to use `vi` (or a similar editor like `nano`) or `echo`. For example, `vi example.txt` will create and open a file called "example.txt". It will be empty until you edit it and save it. More on that later. For now, let's start with
```bash
echo "Hello, world!" > example.txt`.
```
This command prints "Hello, world!" and writes it to the file "example.txt". That's it! You have created your first file!

Now, most files we work with will be generated during the course of our analyses. If this is the result of a unix pipeline, then it will be created from stout using `>`. If it is from a program that is written to work within a unix environment (but is not a unix progrom itself), then it will be genereated using `-o`, `-out`, or `-O` flags.

## Viewing files

Sometimes we need to see all or parts of a file. Here are some useful commands for doing so:
- head: allows one to see n lines at the start of a file where `-n` defines how many; default is 10
- tail: allows one to see n lines at the start of a file where `-n` defines how many; default is 10
- less: allows one to scroll through a file
- more: allows one to scroll through a file faster
- cat: the whole file is read to stout at once

## Manipulating files

There are several ways that a file can be manipulated. Here is an example of how to append the output of a command to an existing file, written as an annotated bash script that I then embedded within this GitHub page. One can copy and paste this directly into terminal and run it.
```bash
#First, we are going to create a new example.txt file. If you created the one earlier, don't worry, this will overwrite it.

echo "Hello, starshine!" > example.txt

#Sometimes, we want to add to a file. We can do this by using >>

echo "The Earth says Hello!" >> example.txt

#This should create a new line within example.txt with "The Earth says Hello!"
#We can check that it works with head

head example.txt
```

We are going to move on to some more complicated means of manipulating files. Thus, you will need to download the [example_data.txt](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/2021_Bio_Info_Bootcamp/Unix_in_a_nutshell/example_data.txt) file. It wil be better to download it from the Google Drive folder rather than from GitHub.(As a sidenote, this example dataset is *not* [tidy](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html).) First, let's take a look at the file. For this file we could simply use cat and see the whole file at once. For large files, that usually is not practical. So, we will use head to determine what columns we are working with.
```bash
head -n 1 example_data.txt
```
This should show you the column names of `Baby_name Sex Birth_mass_kg`.Let's say we want to know how many of each kind of observation is in each column. For example, how many of each "Sex" do we have? We can do this by using the `|` to pipe together several unix commands. In the example below, I have a [useless use of cat](https://en.wikipedia.org/wiki/Cat_(Unix)#Useless_use_of_cat). Hey! I am not perfect and none of you should strive to write perfect code! It is a useless use of cat, but it is convient and the code works. To do what we need to do, we must let things like that go.

Anyway, back to the task at hand. How do we determine how many of each "Sex" there are? We are going to do this by reducing the dataset to just the "Sex" column with `cut` and then we are going to use `sort` and `uniq` to count how many are of each state.
```bash
cat example_data.txt | cut -f 2 | sort | uniq -c
```
Here `cut` cuts the dataset to only the column we want. We specify the column by its number using the `-f` flag. Another flag to be aware of with `cut` is `-d`, which tells `cut` what character separates the columns. By default it is tab. With `|` we route the stout into the next command `sort`, which arranges everything in alphabetical order. We pipe that stout into `uniq` that reduces the data to just unique values, and the `-c` flag tells `uniq` that we want a count of how many times each value appears. This is what we should get.
```bash
 6 F
 1 female
 2 M
 1 male
 1 Sex
```
Well, that is interesting. There are two different coding systems being used! That is not ideal. We want the same coding system to be used for all the observations. We can make that happen using unix commands! The simplest way to do this would be to use `tr`, which means "translate". It turns a character or set of characters into a different set. It is entirely approriate to use here, like so.
```bash
cat example_data.txt | tr "male" "M" | tr "female" "F" > tidy_data.txt
```
However, a downside with using `tr` is that we have to create a new file. Maybe we don't want to. Another downside is that `tr`, while useful, is limited in what it can do. A much more powerful tool is `sed`. [sed](https://www.gnu.org/software/sed/manual/sed.html) stands for stream editor, and it allows us to edit the file directly. There is a wonderful [sed tutorial](https://www.grymoire.com/Unix/Sed.html) that I frequently consult and highly reccomend.
```bash
sed -i 's/female/F/g' example_data.txt

sed -i 's/male/M/g' example_data.txt
```
Here the `-i` flag tells `sed` we want to edit within the file directly and not create a new file. The `'s/male/M/g'` tells `sed` that we want to substitute ('s/) "male" for "M" globally (/g'), or throughout the whole file. Let's make sure it worked with `cat example_data.txt`. We should see
```bash
Baby_name       Sex     Birth_mass_kg
Catalina        F       3.5
Hannah  F       3.5
Izy     F       3.5
Jonathan        M       3.5
Megan   F       3.5
Melia   F       3.5
Prashant        M       3.5
Teresa  F       3.5
Trixie  F       3.5
William M       4.5
```
What if we wanted everything to be "male" and "female" instead of "M" and "F"? We could try
```bash
sed -i 's/M/male/g' example_data.txt

sed -i 's/F/female/g' example_data.txt
```
Let's check with `cat example_data.txt`.
```bash
Baby_name       Sex     Birth_mass_kg
Catalina        female  3.5
Hannah  female  3.5
Izy     female  3.5
Jonathan        male    3.5
maleegan        female  3.5
maleelia        female  3.5
Prashant        male    3.5
Teresa  female  3.5
Trixie  female  3.5
William male    4.5
```
Uh oh! It replaced the "M"'s in the names with "male"! We did not want that! Here too is where `sed` is more powerful than `tr`. With `sed` we can fine tune the pattern that we want replaced. Since all the "M"'s that we do want replaced are preceded (and followed by) a tab, we can use that. Here, we are briefly going to brush against [regular expressions](https://www.grymoire.com/Unix/Regular.html). In unix, we can represent a tab with `\t`, which is a regular expression.
```bash
sed -i 's/\tM/\tmale/g' example_data.txt

sed -i 's/\tF/\tfemale/g' example_data.txt
```
Let's check to see if this fixes the problem. `cat example_data.txt`.
```bash
Baby_name       Sex     Birth_mass_kg
Catalina        female  3.5
Hannah  female  3.5
Izy     female  3.5
Jonathan        male    3.5
Megan   female  3.5
Melia   female  3.5
Prashant        male    3.5
Teresa  female  3.5
Trixie  female  3.5
William male    4.5
```

Let's ask another question: How many babies of each mass were born?
```bash
cat example_data.txt | cut -f 3 | sort | uniq -c
```
We should get
```bash
9 3.5
1 4.5
1 Birth_mass_kg
```
4.5kg!? Wow, that is a big baby! With this dataset, figuring out which baby has a mass of 4.5kg is easy. We would simply `cat` the file and figure it out. But if we had hundreds of enteries in the file, that approach would not be feasible. Instead, we could pull that line out of file with `grep`. [Grep](https://en.wikipedia.org/wiki/Grep) is an acronym for "Globally search for Regular Expression and Print matching lines"; however, I think of it as a [corruption](http://www.artandpopularculture.com/Corruption_%28linguistics%29) of the word "grab". Thus, `grep` grabs and prints what we ask for.
```bash
cat example_data.txt | grep "4.5"
```
We see that William is the big baby!

`awk` is a much more powerful tool than `sed`, but I do not know to use it, per se. Of the times that I have [used awk](https://github.com/Michigan-Mycology/Lab-Code-and-Hacks/blob/master/Phylogenomics/processing_genbank_files/awk_multi_to_single_loop.sh), I simply copied and pasted what someone else wrote. [Here is a tutorial](https://www.grymoire.com/Unix/Awk.html).

And that is it. Those are the main tools that I use to create and manipulate files.
