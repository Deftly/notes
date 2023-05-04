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
The `ls` command lists the contents of a directory. The default is the current directory but you can add any directory or file as an argument. It also has many useful options like `ls -l` for a detailed listing and `ls -f` to display file type information.

```Shell
$ ls -l 
total 4
drwxr-xr-x. 1 siddharth siddharth   6 May  4 08:57 aws_certs
drwxr-xr-x. 1 siddharth siddharth 438 May  4 08:57 git
drwxr-xr-x. 1 siddharth siddharth  22 May  4 08:57 Go
drwxr-xr-x. 1 siddharth siddharth  52 May  4 08:57 Linux
-rw-r--r--. 1 siddharth siddharth   8 May  4 08:57 README.md
drwxr-xr-x. 1 siddharth siddharth  16 May  4 08:57 Rust
```

### 2.3.2 `cp`
In its simplest form, `cp` copies files, for example to copy *file1* to *file2*:
```Shell
$ cp file1 file2
```
You can also copy a file to another directory, keeping the same file name in that directory: 
```Shell
$ cp file1 dir
```
To copy more than one file to a directory named *dir*:
```Shell
$ cp file1 file2 file3 dir
```

### 2.3.3 `mv`
The `mv`(move) command works much like `cp`. In its simplest form it renames a file.
```Shell
$ mv file1 file2
```
You can also use `mv` to move files to other directories the same way as `cp`.

### 2.3.4 `touch`
The `touch` command can create a file. If the target file already exists, `touch` doesn't change the file, but it does update the file's modification timestamp.
```Shell
$ touch file
```

### 2.3.5 `rm`
The `rm` command deletes(removes) a file. After you remove a file, it's usually gone from your system and generally cannot be undeleted unless you restore it from a backup.
```Shell
$ rm file
```

### 2.3.6 `echo`
The `echo` command prints its arguments to the standard output:
```Shell
$ echo Hello again.
Hello again.
```
The `echo` command very useful for finding expressions of shell globs(wildcards such as `*`) and variables(such as `$HOME`) which we will see later in this section.

## 2.4 Navigating Directories

### 2.4.1 `cd`

### 2.4.2 `mkdir`

### 2.4.3 `rmdir`

### 2.4.4 Shell Globbing ("Wildcards")

## 2.5 Intermediate Commands

### 2.5.1 `grep`

### 2.5.2 `less`

### 2.5.3 `pwd`

### 2.5.4 `diff`

### 2.5.5 `file`

### 2.5.6 `find` and `locate`

### 2.5.7 `head` and `tail`

### 2.5.8 `sort`

## 2.6 Changing Your Password and Shell

## 2.7 Dot Files

## 2.8 Environment and Shell Variables 

## 2.9 The Command Path

## 2.10 Special Characters

## 2.11 Command-Line Editing

## 2.12 Text Editors

## 2.13 Getting Online Help

## 2.14 Shell Input and Output

### 2.14.1 Standard Error

### 2.14.2 Standard Input Redirection

## 2.15 Understanding Error Messages

### 2.15.1 Anatomy of a Unix Error Message

### 2.15.2 Common Errors

## 2.16 Listing and Manipulating Processes

### 2.16.1 Command Options

### 2.16.2 Process Termination

### 2.16.3 Job Control

### 2.16.4 Background Processes

## 2.17 File Modes and Permissions

### 2.17.1 Modifying Permissions

### 2.17.2 Working with Symbolic Links

## 2.18 Archiving and Compressing Files

### 2.18.1 `gzip`

### 2.18.2 `tar`

### 2.18.3 Compressed Archives (.tar.gz)

### 2.18.4 `zcat`

### 2.18.5 Other Compression Utilities

## 2.19 Linux Directory Hierarchy Essentials

### 2.19.1 Other Root Subdirectories

### 2.19.2 The */usr* Directory

### 2.19.3 Kernel Location

## 2.20 Running Commands as the Superuser

### 2.20.1 `sudo`

### 2.20.2 */etc/sudoers*

### 2.20.3 sudo Logs

## 2.21 Looking Forward



















