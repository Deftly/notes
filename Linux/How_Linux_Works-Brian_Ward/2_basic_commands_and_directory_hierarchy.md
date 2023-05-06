# Basic Command And Directory Hierarchy
<!--toc:start-->
- [Basic Command And Directory Hierarchy](#basic-command-and-directory-hierarchy)
  - [2.1 The Bourne Shell: /bin/sh](#21-the-bourne-shell-binsh)
  - [2.2 Using the Shell](#22-using-the-shell)
    - [2.2.1 The Shell Window](#221-the-shell-window)
    - [2.2.2 cat](#222-cat)
    - [2.2.3 Standard Input and Standard Output](#223-standard-input-and-standard-output)
  - [2.3 Basic Commands](#23-basic-commands)
    - [2.3.1 ls](#231-ls)
    - [2.3.2 cp](#232-cp)
    - [2.3.3 mv](#233-mv)
    - [2.3.4 touch](#234-touch)
    - [2.3.5 rm](#235-rm)
    - [2.3.6 echo](#236-echo)
  - [2.4 Navigating Directories](#24-navigating-directories)
    - [2.4.1 cd](#241-cd)
    - [2.4.2 mkdir](#242-mkdir)
    - [2.4.3 rmdir](#243-rmdir)
    - [2.4.4 Shell Globbing ("Wildcards")](#244-shell-globbing-wildcards)
  - [2.5 Intermediate Commands](#25-intermediate-commands)
    - [2.5.1 grep](#251-grep)
    - [2.5.2 less](#252-less)
    - [2.5.3 pwd](#253-pwd)
    - [2.5.4 diff](#254-diff)
    - [2.5.5 file](#255-file)
    - [2.5.6 find and locate](#256-find-and-locate)
    - [2.5.7 head and tail](#257-head-and-tail)
    - [2.5.8 sort](#258-sort)
  - [2.6 Changing Your Password and Shell](#26-changing-your-password-and-shell)
  - [2.7 Dot Files](#27-dot-files)
  - [2.8 Environment and Shell Variables](#28-environment-and-shell-variables)
  - [2.9 The Command Path](#29-the-command-path)
  - [2.10 Special Characters](#210-special-characters)
  - [2.11 Command-Line Editing](#211-command-line-editing)
  - [2.12 Text Editors](#212-text-editors)
  - [2.13 Getting Online Help](#213-getting-online-help)
  - [2.14 Shell Input and Output](#214-shell-input-and-output)
    - [2.14.1 Standard Error](#2141-standard-error)
    - [2.14.2 Standard Input Redirection](#2142-standard-input-redirection)
  - [2.15 Understanding Error Messages](#215-understanding-error-messages)
    - [2.15.1 Anatomy of a Unix Error Message](#2151-anatomy-of-a-unix-error-message)
    - [2.15.2 Common Errors](#2152-common-errors)
  - [2.16 Listing and Manipulating Processes](#216-listing-and-manipulating-processes)
    - [2.16.1 Command Options](#2161-command-options)
    - [2.16.2 Process Termination](#2162-process-termination)
    - [2.16.3 Job Control](#2163-job-control)
    - [2.16.4 Background Processes](#2164-background-processes)
  - [2.17 File Modes and Permissions](#217-file-modes-and-permissions)
    - [2.17.1 Modifying Permissions](#2171-modifying-permissions)
    - [2.17.2 Working with Symbolic Links](#2172-working-with-symbolic-links)
  - [2.18 Archiving and Compressing Files](#218-archiving-and-compressing-files)
    - [2.18.1 gzip](#2181-gzip)
    - [2.18.2 tar](#2182-tar)
    - [2.18.3 Compressed Archives (.tar.gz)](#2183-compressed-archives-targz)
    - [2.18.4 zcat](#2184-zcat)
    - [2.18.5 Other Compression Utilities](#2185-other-compression-utilities)
  - [2.19 Linux Directory Hierarchy Essentials](#219-linux-directory-hierarchy-essentials)
    - [2.19.1 Other Root Subdirectories](#2191-other-root-subdirectories)
    - [2.19.2 The /usr Directory](#2192-the-usr-directory)
    - [2.19.3 Kernel Location](#2193-kernel-location)
  - [2.20 Running Commands as the Superuser](#220-running-commands-as-the-superuser)
    - [2.20.1 sudo](#2201-sudo)
    - [2.20.2 /etc/sudoers](#2202-etcsudoers)
    - [2.20.3 sudo Logs](#2203-sudo-logs)
  - [2.21 Looking Forward](#221-looking-forward)
<!--toc:end-->

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

### 2.3.1 ls
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

### 2.3.2 cp
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

### 2.3.3 mv
The `mv`(move) command works much like `cp`. In its simplest form it renames a file.
```Shell
$ mv file1 file2
```
You can also use `mv` to move files to other directories the same way as `cp`.

### 2.3.4 touch
The `touch` command can create a file. If the target file already exists, `touch` doesn't change the file, but it does update the file's modification timestamp.
```Shell
$ touch file
```

### 2.3.5 rm
The `rm` command deletes(removes) a file. After you remove a file, it's usually gone from your system and generally cannot be undeleted unless you restore it from a backup.
```Shell
$ rm file
```

### 2.3.6 echo
The `echo` command prints its arguments to the standard output:
```Shell
$ echo Hello again.
Hello again.
```
The `echo` command very useful for finding expressions of shell globs(wildcards such as `*`) and variables(such as `$HOME`) which we will see later in this section.

## 2.4 Navigating Directories
The Unix directory hierarchy starts at */* and is called the *root directory*. The directory separator is the slash(/). There are several standard subdirectories in the root directory like */usr* which we'll cover in [section 2.19](#219-linux-directory-hierarchy-essentials)

When you refer to a file or directory, you specify a *path* or *pathname*. When a path starts with */* it's a *full* or *absolute path*. A path component identified by two dots(..) specifies the parent of a directory and one dot(.) refers to the current directory. A path not beginning with */* is called a *relative path*.

### 2.4.1 cd
The *current working directory* is the directory that a process(such as the shell) is currently in. Each process can independently set its own current working directory, the `cd` command changes the shell's current working directory:
```Shell
$ cd dir
```

If you omit *dir*, the shell returns to your *home directory*, the directory where you started when you first logged in, this is often abbreviated with the *~* symbol.

> **_NOTE:_** The `cd` command is a shell built-in. It wouldn't work as a separate program because if it were to run as a subprocess, it could not(normally) change its parent's current working directory.

### 2.4.2 mkdir
The `mkdir` command creates a new directory *dir*:
```Shell
$ mkdir dir
```

### 2.4.3 rmdir
The `rmdir` command removes the directory *dir*:
```Shell
$ rmdir dir
```

If *dir* isn't empty, this command fails. To quickly delete a non-empty directly instead use `rm -r dir`, this will delete the directory and all its contents. Be careful when using this command as it can do serious damage especially if run as the superuser. The `-r` option specifies *recursive delete*, always double check your command before you run it.

### 2.4.4 Shell Globbing ("Wildcards")
The shell can match simple patterns to file and directory names, a process known as *globbing*. The simplest of these is the glob character(*), which tells the shell to match any number of arbitrary characters. The following prints a list of all the files in the current directory:
```Shell
$ echo *
```

The shell matches arguments containing globs to filenames, substitutes those filenames for those arguments, and then runs the revised command. This substitution is called *expansion*, here are some examples:
- *at\** expands to all filenames that start with *at*
- *\*at* expands to all filenames that end with *at*
- *\*at\** expands to all filenames that contain *at*

If no files match a glob, the `bash` shell performs no expansion, and the command runs with the literal characters.

Another shell glob character, the question mark(?), instructs the shell to match exactly one arbitrary character. For example, *b?at* matches *boat* and *brat*.

If you don't want the shell to expand a glob in a command enclose the glob in single quotes(''). For example, the command `echo '*'` prints a star.

There is more to the shell's patter-matching capabilities, but *\** and *?* are the basics you need to know.

## 2.5 Intermediate Commands
This section covers the most essential intermediate Unix commands.

### 2.5.1 grep
The `grep` command prints the lines from a file or input stream that match an expression. For example, to print the lines in the */etc/passwd* file that contains the text *root* use this:
```Shell
$ grep root /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```

Two of the most important `grep` options are `-i`(for case-insensitive matches) and `-v`(which inverts the search and prints all lines that *don't* match)

`grep` understands *regular expressions*, which are more powerful that wildcard-style patterns, and have a different syntax.

> **_NOTE:_** To learn more you can read Mastering Regular Expressions, 3rd edition, by Jeffery E. F. Friedl or other online documentation on regular expressions.

### 2.5.2 less
The `less` command comes in handy when a file is really big or when a command's output is long and scrolls off the top of the screen. To page through a big file like */usr/share/dict/words* use `less /usr/share/dict/words`. When running `less` you'll see the contents of the file one screenful at a time. Press the spacebar to go forward in the file and press b(lowercase) to skip back one screenful, and press q to quit.

> **_NOTE:_** The `less` command is an enhanced version of an older program named `more`. Linux desktops and servers have `less` but it's not standard on many embedded systems and other Unix systems. If you can't run `less` try `more`.

You can also search for text inside `less`. To search forward you can type */word* and to search backward you can use *?word*. When you find a match press n to jump to the next matching occurrence of the word.

We'll see in [section 2.14](#214-shell-input-and-output) that you can send the standard output of nearly any program directly to another program's standard input. This is very useful when you have a command with a lot of output to sift through, here's an example:
```Shell
$ grep ie /usr/share/dict/words | less
```

### 2.5.3 pwd
The `pwd`(print working directory) command simply outputs the name of the current working directory. This might not seem necessary since most Linux distributions have user accounts with the current working directory in the prompt, but there are two reasons why it is needed. First, not all prompts include the current working directory. Second, symbolic links, which we'll cover in [section 2.17.2](#2172-working-with-symbolic-links) can sometime obscure the true full path of the current working directory. We can use `pwd -P` to remove this confusion.

### 2.5.4 diff
To see the differences between two text files use `diff`:
```Shell
$ diff file1 file2
```

There are several options to control the format of the output, though the default is often the most comprehensible for humans. However, most programmers use `diff -u` when sending the output to others as automated tools have an easier time with this format

### 2.5.5 file
If you see a file and are unsure of its format, try using the `file` command to see if the system can guess it:
```Shell
$ file file_name
```

### 2.5.6 find and locate
It can be frustrating when you know that a certain file is in a directory tree but you don't know exactly where. Run `find` to find *file* in *dir* as follows:
```Shell
$ find dir -name file -print
```

Most systems also have a `locate` command for finding files. Rather than searching for a file in real time, `locate` searches an index that the system builds periodically. Searching with `locate` is much faster than `find`, but if the file you're looking for is newer than the index, `locate` won't find it.

### 2.5.7 head and tail
The `head` and `tail` commands allow you to quickly view a portion of a file or stream of data. For example, `head /etc/passwd` shows the first 10 lines of the password file and `tail /etc/passwd` shows the last 10 lines. To change the number of lines to display use the `-n` option, where *n* is the number of lines you want to see(`head -5 /etc/passwd`).

### 2.5.8 sort
The `sort` command quickly puts the lines of a text file in alphanumeric order. If the file's lines start with numbers and you want to sort in numerical order, use the `-n` option. The `-r` option reverse the order of the sort.

## 2.6 Changing Your Password and Shell
Use the `passwd` command to change your password. You'll be prompted for your old password and then prompted for your new password twice. The best passwords tends to be long "nonsense" sentences that are easy to remember, try to aim for 16 characters or more.

You can change your shell with the `chsh` command(some other shells are `zsh`, `ksh`, or `tcsh`), but keep in mind these notes all assume the use of `bash` and some of the examples may not work with other shells.

## 2.7 Dot Files
Dot files are configuration files and directories whose names begin with a dot(.), and are hidden so if you run just `ls` you won't see them listed. To list them use `ls -a`. Some common dot files are *.bashrc* and *.gitconfig*, there are also dot directories such as *.ssh*.

There's nothing special about dot files or directories other than some programs don't show them by default in order to reduce clutter. Additionally, shell globs don't match dot files unless you explicitly use a patter such as `.*`.

## 2.8 Environment and Shell Variables 
The shell can store temporary variables, called *shell variables*, containing the values of text strings. They are useful for keeping track of values in scripts, and some shell variables control the way the shell behaves(For example, the `bash` shell reads the `PS1` variable before displaying the prompt). 

To assign a value to a shell variable, use the equal sing(=) like below:
```Shell
$ STUFF=blah
```

The preceding example sets the value of the variable named `STUFF` to `blah`. To access this variable, use `$STUFF`(for example, try `echo $STUFF`). We'll cover the many uses of shell variables in [section 11](./11_introduction_to_shell_scripts.md)

> **+_NOTE:_** Don't put any spaces around the `=` when assigning a variable.

An *environment variable* is like a shell variable but it's not specific to the shell. All processes on Unix systems have environment variable storage. The main difference between environment and shell variables is that the operating system passes all of your shell's environment variables to programs that the shell runs, whereas shell variables cannot be accessed in commands that you run.

You assign an environment variable with the shell's `export` command like so:
```Shell
$ STUFF=blah
$ export STUFF
```

Because child processes inherit environment variables from their parent, many programs read them for configuration and options. For example, you can put your favorite `less` command-line options in the `LESS` environment variable, and `less` will use those options when you run it.(Many manual pages contain a section labeled ENVIRONMENT that describes these variables.)

## 2.9 The Command Path
`PATH` is a special environment variable that contains the *command path* or just *path* for short. It is a list of system directories that the shell searches when trying to locate a command. For example, when you run `ls`, the shell searches the directories listed in `PATH` for the `ls` program. If programs with the same name appear in several directories in the path, the shell runs the first matching program.

If your run `echo $PATH`, you'll see the path components are separated by colons(:).
```Shell
$ echo $PATH
/home/siddharth/.cargo/bin:/home/siddharth/.local/bin:/home/siddharth/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/usr/local/go/bin:/usr/local/go/bin:/home/siddharth/.local/share/nvim/mason/bin:/usr/local/go/bin
```

You can tell the shell to look in more places for programs by changing the `PATH` variable like so:
```Shell
$ PATH=$PATH:dir
```

> **_NOTE:_** You can accidentally wipe out your entire path if you mistype `$PATH` when modifying your path. However this damage isn't permanent as you can just start a new shell. To create lasting effects you need to mistype when editing certain configuration files and even then it still isn't too difficult to rectify.

## 2.10 Special Characters
When discussing Linux with others, you should know some of the common names for special characters that you will encounter.

| Characters  | Name(s)   | Uses   |
|-------------- | -------------- | -------------- |
| * | star, asterisk  | Regular expression, glob character  |
| . | dot | Current directory, file/hostname delimiter |
| ! | bang | Negation, command history |
| \| | pipe | Command pipes |
| / | (forward)slash | Directory delimiter, search command |
| \ | backslash | Literals, macros (never directories) |
| $ | dollar | Variables, end of line |
| ' | tick, (single)quote | Literal strings |
| ` | backtick, backquote | Command substitution |
| " | double quote | Semi-literal strings |
| ^ | caret | Negation, beginning of line | 
| ~ | tilde, squiggle | Negation, directory shortcut |
| # | hash, sharp, pound | Comments, preprocessor, substitutions |
| [ ] | (square)brackets | Ranges |
| { } | brace, (curly)brackets | Statement blocks, ranges |
| _ | underscore, under | substitute for space used when spaces aren't wanted or allowed |

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

### 2.18.1 gzip

### 2.18.2 tar

### 2.18.3 Compressed Archives (.tar.gz)

### 2.18.4 zcat

### 2.18.5 Other Compression Utilities

## 2.19 Linux Directory Hierarchy Essentials

### 2.19.1 Other Root Subdirectories

### 2.19.2 The /usr Directory

### 2.19.3 Kernel Location

## 2.20 Running Commands as the Superuser

### 2.20.1 sudo

### 2.20.2 /etc/sudoers

### 2.20.3 sudo Logs

## 2.21 Looking Forward

