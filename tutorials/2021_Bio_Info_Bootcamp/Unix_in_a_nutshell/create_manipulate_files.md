# Creating and manipulating files

Several types of files exist and, usually, the kind we will be dealing with are [regular files](https://en.wikipedia.org/wiki/Unix_file_types). Regualr files come in several flavors that are usually denoted by their [filename extension](https://en.wikipedia.org/wiki/Filename_extension). This is where one of the key features of unix comes into play: Most of the files we will be working with will be some form of a text file. The "simplest" is a [plain text file](https://en.wikipedia.org/wiki/Plain_text), but this also includes [vcf files](https://samtools.github.io/hts-specs/VCFv4.2.pdf) and [fasta files](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=BlastHelp). Remember, unix is built around manipulating text files, which means any command that will work on a vcf file will work equally well on a fasta file or any other kind of text file. Thus, unix has near universal compatability and be applied to many of our bioinformatic needs.

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

Now, most files we work with will be generated during the course of our analyses. If this is the result of a unix pipeline, then it will be created from stout using `>`. If it is from a program that is written to work within a unix environment (but is not a unix progrom itself), then it will like be genereated using `-o`, `-out`, or `-O` flags.

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

We are going to move on to some more complicated means of manipulating files. Thus, you will need to download the [example_data.txt](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/2021_Bio_Info_Bootcamp/Unix_in_a_nutshell/example_data.txt) file. (As a sidenote, this example dataset is *not* [tidy](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html).) First, let's take a look at the file. For this file we could simply use cat and see the whole file at once. For large files, that usually is not practical. So, we will use head to determine what columns we are working with.
```bash
head -n 1 example_data.txt
```
This should show you the column names of `Baby_name eye_color Sex Birth_mass_kg`.Let's say we want to know how many of each kind of observation is in each column. For example, how many of each "Sex" do we have? We can do this by using the `|` to pipe together several unix commands. In the example below, I have a [useless use of cat](https://en.wikipedia.org/wiki/Cat_(Unix)#Useless_use_of_cat). Hey! I am not perfect and none of you should strive to write perfect code! It is a useless use of cat, but it is convient and the code works. To do what we need to do, we must let things like that go.

Anyway, back to the task at hand. How do we determine how many of each "Sex" there are? We are going to do this by reducing the dataset to just the "Sex" column with `cut` and then we are going to use `sort` and `uniq` to count how many are of each state.
```bash
cat example_data.txt | cut -f 3 | sort | uniq -c
```
Here `cut` cuts the dataset to only the column we want. We specify the column by its number using the `-f` flag. Another flag to be aware of with `cut` is `-d`, which tells `cut` what character separates the columns. By default it is tab. With `|` we route the stout into the next command `sort`, which arranges everything in alphabetical order. We pipe that stout into `uniq` that reduces the data to just unique values, and the `-c` flag tells `uniq` that we want a count of how many times each value appears. This is what we should get.
```bash
F 6
M 2
female  1
male  1
```
Well, that is interesting. There are two different coding systems being used! That is not ideal. We want the same coding system to be used for all the observations. We can make that happen using unix commands! 





