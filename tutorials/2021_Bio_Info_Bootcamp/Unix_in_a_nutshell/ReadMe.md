# Unix in a nutshell

**Learning Objectives**

2.[How to navigate within an unix/unix-like system](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/2021_Bio_Info_Bootcamp/Unix_in_a_nutshell/navigating_unix.md)

3.[How to create and manipulate files within an unix/unix-like system](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/2021_Bio_Info_Bootcamp/Unix_in_a_nutshell/create_manipulate_files.md)

4.[How to download/install within an unix/unix-like system]()

5.[How to set environmental variables and edit the bash profile within an unix/unix-like system](https://github.com/wjdavis90/Omics_lab_server/blob/main/tutorials/setting_PATH.md)

[Unix](https://en.wikipedia.org/wiki/Unix) (and unix-like systems) is a poweful, flexible operating system. Its [philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) is that programs (aka, commands) should be built to do a single job and do it well; work well together; work with text format; and, be "easy" to use. 

I keep a running list of [useful Unix commands](https://github.com/wjdavis90/Omics_lab_server/blob/main/useful_unix_commands). I add try to add new commands to the list as I learn them. If you know any thing I do not, feel free to add them! As well, some of the items on the list are notations rather than commands, such as `#`, which is used to indicate that a line is a comment rather than a command to be executed. As you can see, there are not that many. I tend to use the same commands over and over again to accomplish what I need. Here are the commands that I most freqently use, and that we will use during bootcamp.

- cd  change directory (folder)
- mkdir create directory
- pwd print path of current working directory
- vi  text editor
- cp  copy
- head/tail view top/bottom of a file
- cat concatenate or merge files
- more/less scroll through a file
- grep  grab regular expressions/patterns from a file
- cut cut the file by columns
- sort  sort the file by columns
- uniq  reveal only the unique values in a list
- tr  translate a set of characters to another set; like find and replace
- sed a magic tool
- awk an even more powerful magic tool
- man view the help page of a command
- wget  get a file from the internet
- git git a folder from GitHub
- locate  find something
- which tells you what version you are using
- chmod change/modify permissions

For any command in unix, one can view its help page. For example, `man grep` will show you the help page for grep.

