# Using Essential Tools
The following RHCSA exam objectives are covered in this chapter:
- Use input-output redirection(>,>>,|,2>,etc.)
- Access a shell prompt and issue commands with correct syntax
- Create and edit text files 
- Locate, read, and use system documentation including `man`, `info`, and files in /usr/share/doc

## Do I Know This Already?
- What command enables you to redirect standard output as well as standard error to a file?
    - `> file 2>&1`

- You want to set a local variable that will be available for every user in every shell, in what file should you set the variable?
    - /etc/bashrc is processed when a subshell is started, and it is included while starting a login shell as well.

- A user created a script with the name myscript and verifies that the script permissions are set as executable, but are unable to run it using the command `myscript`. What is the most likely explanation?
    - The directory that contains the script is not in the $PATH variable.

- You need the output of the command `ls` to be used as input for `less`, how do you do this?
    - `ls | less`

- A user by accident has typed a password which now shows as item 299 in history. What can you do to ensure the password is not stored in history?
    - `history -d 299`

- Which command lets you replace every occurrence of *old* with *new* in a text file using `vim`?
    - **:%s/old/new/g**, **%** means that it must be applied on the entire file. The **s** stands for substitute. The **g** option is used to apply the command to not only the first occurrence on a line but all occurrences in the line. 

- How do you show a message to all users who have just logged in to a shell session on your server?
    - To show a message after a user has logged in put the message in /etc/motd. If you want the message to display before the user logs in edit the /etc/issue file.

- You true using `man -k user` but you get the message "nothing appropriate", what can you do to try and fix this?
    - You can use `sudo mandb` to update the mandb database. On RHEL 6 and earlier the `makewhatis` command is used instead

## Foundation Topics

### Basic Shell Skills
The shell is the default working environment for a Linux admin. There are different available shells for Linux but Bash is the most common. 

#### Understanding Commands
Typically command syntax has three basic parts: the command, its options, and its arguments. 

Let's look at the `ls` command as an example, this command shows a list of files in the current directory. Options are a part of the program code and they modify what the command does. For instance, when you use the `-l` option with `ls`, a long listing of filenames and properties is displayed.

Arguments refer to anything that the command addresses, so anything you put after the command is an argument(including options). Apart from options, you can have other arguments which serve as a target for the command. The command `ls -l /etc` has two arguments `-l` and `/etc`. The first modifies the behavior of the command and the second specifies where the target should do its work.

#### Executing Commands
The purpose of the Linux shell is to provide an environment in which commands can be executed and it makes a distinction between three kinds of commands:
- Aliases
- Internal commands/shell built-ins
- External commands

An *alias* is a command that user can define as needed. To define an alias, use `alias newcommand='oldcommand'`. Aliases are executed before anything else. So, if you have an alias with the name `ll` but also a command with the name `ll`, the alias will always take precedence for the command unless a complete pathname is used.

An internal command or shell built-in is part of the shell itself and doesn't have to be loaded from disk separately. An external command is a command that exists as an executable file on the disk of the computer. When a user executed a command, the shell first looks to determine whether it is a built-in. If it isn't it looks for an executable file with a name that matches the command on disk. To check whether a command is a Bash internal or an executable file you can use the `type` command.

The external commands are found by the shell using the $PATH variable. This variable defines a list of directories that are searched for a matching filename when a user enters a command. The $PATH variable can be set for specific users but in general most users will be using the same $PATH variable. The exception is the root user, who needs access to specific administration commands.

#### I/O Redirection
By default, when a command is executed it shows its results on the command prompt, also referred to as STDOUT. The shell also has a default destination for error messages(STDERR) and to accept input(STDIN).

| Name | Default Destination | Use in Redirection | File Descriptor Number |
|---------------- | --------------- | --------------- | --------------- |
| STDIN | Computer keyboard    | < (same as 0<)    | 0    |
| STDOUT | Computer monitor   | > (same as 1>)   | 1   |
| STDERR | Computer monitor  | 2>   | 2   |

Programs started from the command line have no idea what they are reading from or writing to. They just read from what the Linux kernal calls file descriptor 0 if they want to read from standard input, and they write to file descriptor number 1 to display non-error output and to file descriptor 2 if they have error messages to be output. Using redirection symbols such as <,>, and |, the shell can connect the file descriptors to files or other commands.

With I/O redirection, files can be used to replace the default STDIN, STDOUT, and STDERR. You can also redirect to device files. A device file is a file that is used to access a specific hardware component, for instance your hard disk. Note that to access most device files, you need to have root privileges.


## Understanding the Shell Environment

## Finding Help

## Summary

## Exam Preparation Tasks

## Review All Key Topics

## Complete Tables and Lists from Memory

## Define Key Terms

## Review Questions

## End-of-Chapter Lab
