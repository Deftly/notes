# Basic Command And Directory Hierarchy
This section will cover the Unix commands and utilities you'll frequently encounter. You might ask why Unix commands? This is because Linux is a Unix flavor at heart, and you can use all these commands on BSD and other Unix-flavored systems. Knowing these commands can also boost your understanding of the kernel as many correspond directly to system calls. 

## 2.1 The Bourne Shell: /bin/sh
A *shell* is a program that runs commands like the ones users enter into a terminal window. These commands can be other programs or built-in features of the shell.

There are many different Unix shells, but all derive features from the Bourne shell(*/bin/sh*), a standard shell developed at Bell Labs for early versions of Unix. Linux uses an enhanced version of the Bourne shell called `bash` or the "Bourne-again" shell. The `bash` shell is the default on most Linux distributions, and */bin/sh* is normally a link to `bash` on a Linux system.

> **_NOTE:_** You can change your shell with the `chsh` command

## 2.2 Using the Shell
When installing Linux you should create at least one regular user to be your personal account.

### 2.2.1 The Shell Window
The easiest way to open up a shell window is to open a terminal application which starts a shell inside a  new window. It should display a prompt at the top that usually ends with a dollar sign($). The prompt should look something like `[*name@host path*]$` where *name* is your username, *host* is the name of your machine, and *path* is your current working directory. Enter the following command and press ENTER:

```Shell
$ echo Hello there.
```

Command usually begin with a program to run and may be followed by *arguments* that tell the program what to operate on and how to do so. Many arguments are options that modify the default behavior of a program and typically begin with a dash(-).

# 2.2.2 cat
The `cat` program simply outputs the contents of one or more files or another source of input. The general syntax is as follows:

```Shell
$ cat file1 file2 ...
```


















