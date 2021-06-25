# How to set programs (or scripts) to $PATH

**Purpose:** This document tells how to permanently set programs to your $PATH variable. It is based off experience and this [tutorial](https://stackabuse.com/how-to-permanently-set-path-in-linux).

**Tip:** If one is going to be using one of these [programs](https://github.com/wjdavis90/Omics_lab_server/blob/main/program_list.md) regularly, it is recommended one edits one's `.bash_profile` file in one's home directory to export it to PATH every time one logs in. 

Example using Dsuite:

This is what `.bash_profile` looks like before editing.

```bash
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
```

Using vi or vim (or nano or emacs), one adds `PATH=$PATH:/scratch/Tools/Dsuite/Build/` and `PATH=$PATH:/scratch/Tools/Dsuite/utils/` to it so that it looks like this.

```bash
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin
PATH=$PATH:/scratch/Tools/Dsuite/Build/
PATH=$PATH:/scratch/Tools/Dsuite/utils/

export PATH
```

NOTE: One may need to set PATH to a subdirectory, such as `/bin/`, `/Build/`, `/utils/`, or `/src/`!

The order that the directories is listed in is important. The system will search the directories in the order that you list them. Usually, this is not a problem, but it can be if multiple versions of a program are installed on a system.

Once you have added the appropriate directories that you want to be in your $PATH, you must run `source ~/.bash_profile` for the the changes to take effect. 
