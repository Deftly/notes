# Basic Command And Directory Hierarchy
This section will cover the Unix commands and utilities you'll frequently encounter. You might ask why Unix commands? This is because Linux is a Unix flavor at heart, and you can use all these commands on BSD and other Unix-flavored systems. Knowing these commands can also boost your understanding of the kernel as many correspond directly to system calls. 

## 2.1 The Bourne Shell: /bin/sh
A *shell* is a program that runs commands like the ones users enter into a terminal window. These commands can be other programs or built-in features of the shell.

There are many different Unix shells, but all derive features from the Bourne shell(*/bin/sh*), a standard shell developed at Bell Labs for early versions of Unix. Linux uses an enhanced version of the Bourne shell called `bash` or the "Bourne-again" shell. The `bash` shell is the default on most Linux distributions, and */bin/sh* is normally a link to `bash` on a Linux system.

> **_NOTE:_** You can change your shell with the `chsh` command

## 2.2 Using the Shell
When installing Linux you should create at least one regular user to be your personal account.

### 2.2.1 The Shell Window
The easiest way to open up a shell window is to open a terminal application which starts a shell inside a  new window. It should display a prompt at the top that usually ends with a dollar sign($). The prompt should look something like `[name@host path]$` where *name* is your username, *host* is the name of your machine, and *path* is your current working directory. Enter the following command and press ENTER:

```Shell
$ echo Hello there.
```

Command usually begin with a program to run and may be followed by *arguments* that tell the program what to operate on and how to do so. Many arguments are options that modify the default behavior of a program and typically begin with a dash(-).

### 2.2.2 cat
The `cat` program simply outputs the contents of one or more files or another source of input. The general syntax is as follows:

```Shell
$ cat file1 file2 ...
```

### 2.2.3 Standard Input and Standard Output
Unix processes use I/O *streams* to read and write data. Streams are very flexible, for example, the source of an input stream can be a file, a device, a terminal window, or even the output stream from another process.

To see an input stream at work, enter `cat`(with no arguments) and press ENTER. This time you won't get any immediate output and you won't get your shell prompt back because `cat` is still running. Now if you type anything and press ENTER `cat` will repeat what you typed. Press CTRL-D on an empty line to terminate `cat` and return to the shell prompt.

The reason `cat` adopts an interactive behavior here has to do with streams. When you don't specify an input filename `cat` reads from the *standard input* stream provided by the Linux kernel rather than a stream connected to a file. In this case, the standard input is connected to the terminal where you run `cat`.

> **_NOTE:_** Pressing CTRL-D on an empty line stops the current standard input entry from the terminal with an EOF(end-of-file) message(and often terminates a program). This is different from CTRL-C, which usually terminates a program regardless of its input or output.

*Standard output* is similar, the kernel gives each process a standard output stream where it can write its output. The `cat` command always writes its output to the standard output which was your terminal in this case.

Standard input and output are often abbreviated as *stdin* and *stdout*. Many commands operate similar to `cat`, if you don't specify an input file, the commands read from stdin. Output is a little different. Some programs(like `cat`) send output only to stdout, but others have the option to send output directly to files.

## 2.3 Basic Commands
Most of the following programs take multiple arguments, and some have loads of options and formats, too many to cover here but we'll go over the basics.

### 2.3.1 `ls`














