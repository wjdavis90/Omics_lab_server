# How to navigate within an unix/unix-like system

The operating systems most people are familiar with (e.g., Windows and Mac), are [GUI](https://en.wikipedia.org/wiki/Graphical_user_interface)s, that is, they use a graphical system to interface between the user and the operating system. You can think of it as a point and click system. Unix, on the other hand, is command line driven--evertyhing has to be typed. The first thing to learn in order to use such a system is how to navigate within it. In a GUI, one simply points and clicks at the directory/folder that one wants to go to or program/file one wants to open. In a Unix system, however, one needs a command line program, or command for short, to do so.

First, let's define some vocabulary.
- directory: this is a location on the machine. [Technically, unix has directories and GUIs have folders.](https://en.wikipedia.org/wiki/Directory_(computing)#Folder_metaphor) However, folders and directories are functionally equivalent, and I will use the terms interchangeably.
- path: this is the location of a directory, a program, or a file; e.g., `/home/wdavis/Tools/`. Note: It is not a physical location! It is merely the location within the directory system.
- home: this is directory in which the users personal files are stored. Most often, this the directory that terminal opens in or that you will be in when logging in. You can read more [here](http://www.linfo.org/home_directory.html)
- root: this is the directory, designated with a `/` that contains all other directories on the machine
- pwd: the command to show what directory you are in
- cd: change directory; aka, move from one directory to another
- mkdir: create a directory
- ./ notation meaning the current directory
- ~ notation meaning the home directory
- ../ notation meaning one directory up

To know where one is at, one uses the command `pwd`. [pwd](https://en.wikipedia.org/wiki/Pwd) stands for "print working directory. The output should look something like `/scratch/wdavis/Philornis_project/`. Typically, everyone starts in the home directory. The home directory usually has a path of the form `/home/` and can be abbreviated as `~/`. Since we are all on different machines, we will all have different home directories. However, I want us all to have the same working directory, which is why I asked that you create the following folder and subfolders:
```bash
2021-BioInfo_Boot_Camp
     Data
        Day1
        Day2
     Tools
     Day1
     Day2
```

If you happen to using a mac, getting to this folder in [Terminal](https://support.apple.com/guide/terminal/open-or-quit-terminal-apd5265185d-f365-44cb-8b09-71a064a42125/mac) is easy. First, in Terminal, type `cd `. (Note there is a space after `cd`.) Then highlight the folder `2021-BioInfor_Boot_Camp` and drag and drop it into Terminal. It should display the path of the directory underlying the folder after `cd `. Hit enter. You have sucessfully moved into the the `2021-BioInfor_Boot_Camp` directory.

If you are using a Windows system, then, I am sorry, unix works a bit differently on them. We will have to figure something out together.

If you are using the server, then you will have to hand type the path to the directory. But not to worry! You can start typing and use tab to finish. For instance, if I want to move into `/scratch/wdavis/Philornis_project/Fst/`, then I would type `cd /sc`, hit tab, and it would complete to read `cd /scratch/`. I would keep doing that until the full path is typed. 
