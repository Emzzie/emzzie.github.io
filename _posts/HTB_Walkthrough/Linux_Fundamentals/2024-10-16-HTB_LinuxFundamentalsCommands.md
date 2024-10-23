---
title: HTB Linux Fundamentals - The Commands
description: A summary of the commands introduced in the Linux Fundamentals Module.
categories: [Walkthrough HTB, Linux Fundamentals]
tags: [Basics, Linux]
pin: false
---

## System info

| Command   | Description                                                                                              |
|-----------|----------------------------------------------------------------------------------------------------------|
| `whoami`  | Displays current username.                                                                                |
| `id`      | Returns user's identity.                                                                                 |
| `hostname`| Sets or prints the name of current host system.                                                          |
| `uname`   | Prints basic information about the operating system name and system hardware.                            |
| `pwd`     | Returns working directory name.                                                                          |
| `ifconfig`| The ifconfig utility is used to assign or to view an address to a network interface and/or configure network interface parameters. |
| `ip`      | Utility to show or manipulate routing, network devices, interfaces, and tunnels.                         |
| `netstat` | Shows network status.                                                                                    |
| `ss`      | Another utility to investigate sockets.                                                                  |
| `ps`      | Shows process status.                                                                                    |
| `who`     | Displays who is logged in.                                                                               |
| `env`     | Prints environment or sets and executes command.                                                         |
| `lsblk`   | Lists block devices.                                                                                     |
| `lsusb`   | Lists USB devices.                                                                                       |
| `lsof`    | Lists opened files.                                                                                      |
| `lspci`   | Lists PCI devices.                                                                                       |

## Navigation

| Command   | Description                                                                                                 |
|-----------|-------------------------------------------------------------------------------------------------------------|
| `pwd`     | Print Working Directory                                                                                     |
| `ls`      | List contents inside folder                                                                                 |
| `ls -l`   | List contents to display more information of the directory and files, such as type and permissions          |
| `ls -la`  | List all files, including hidden ones (starting with .)                                                     |
| `cd`      | Change directory                                                                                            |
| `cd -`    | Change back to previous directory                                                                           |
| `cd ..`   | Move up one directory                                                                                       |

## Working with Files and Directories

| Command   | Description                                                                                                 |
|-----------|-------------------------------------------------------------------------------------------------------------|
| `touch <name>`    | Create a file with given name|
| `mkdir <name>`    | Create a directory with given name|
| `mkdir -p <path>` | Create a directory with the given parent directories|
| `tree <directory>`| View directories as a tree structure |
| `mv <from> <to>`  | Move a file or directory from one directory or file to another. I.e renaming the file or moving to another directory|

## Find files and Directories

| Command   | Description                                                                                                 |
|-----------|-------------------------------------------------------------------------------------------------------------|
| `which <name>` | Returns path to the file or link that should be executed. |
| `find <location> <options>`    | Searches for files and directories based on different conditions. Common options include: <br> `-name [name]`: Search by file or directory name. <br> `-type [type]`: Filter by type (e.g., `f` for file, `d` for directory). <br> `-size [size]`: Search by file size (e.g., `+100M` for files larger than 100MB). <br> `-mtime [days]`: Search for files modified within a specific number of days (e.g., `-mtime -7` for files modified in the last week). <br> `-exec [command]`: Execute a command on matching files (e.g., `-exec rm {} \;` to delete found files). |
| `locate`  | Finds files and directories by searching through a local database of existing files and folders |


## File Descriptors and Redirections

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `2>/dev/null` | Redirect STDERR to /dev/null (to skip seeing them) |
| `>`/`1>` | Redirect STDOUT to instance to the right of ">" |
| `<` | Redirect STDIN to instance to the left of "<" |
| `>>` | Append STDOUT to existing file |
| `<<` | Redirects multi-line input (provided inline) as STDIN to a command until a specified delimiter is reached.|
| `|` | Pipe STDOUT from one program to another. |

## Filter Contents

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `more <file>` | Read file  |
| `less <file>` | Read file (doesn't show file content in terminal after quitting) |
| `head <file>` | Print the first 10 lines of a file |
| `tail <file>` | Print the last 10 lines of a file |
| `sort` | Sort output alphabetically if not specified otherwise |
| `grep <pattern>` | Filter STDOUT according to defined pattern |
| `grep -v <pattern>` | Filter STDOUT and exclude defined pattern |
| `cut -d"<delimiter> -f<position>` | Cut output at the delimiter and show the position specified |
| `tr <char> <rep-char>` | Replace a character with given replacement character |
| `column -t` | Display content i tabular form |
| `awk` | A tool to do text-processing -> Really awesome tool! | 
| `sed` | Stream editor looking for defined patterns and replace them with defined patterns |
| `wc` | Count lines (`-l`) or characters automatically |
| `uniq` | Removes duplicate output lines |

## Permission Management

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `chmod`   | Modify file permissions: <br> `u`= owner, `g`= group, `o`= others, `a`= all users <br> `+` = add permission `-`= remove permission <br> `r`= read, `w`= write, `x`= execute <br> e.g `chmod a+r` |
| `chown <user>:<group> <file/directory>`| Change owner and/or group assignment of a file or directory |


## User Management

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `sudo`	| Execute command as a different user. | 
| `su` | 	The su utility requests appropriate user credentials via PAM and switches to that user ID (the default user is the | superuser). A shell is then executed. | 
| `useradd` | 	Creates a new user or update default new user information. | 
| `userdel` | 	Deletes a user account and related files. | 
| `usermod`  | 	Modifies a user account. | 
| `addgroup` | 	Adds a group to the system. | 
| `delgroup` | 	Removes a group from the system. | 
| `passwd`	| Changes user password. | 

## Package Management

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `dpkg` |	The dpkg is a tool to install, build, remove, and manage Debian packages. The primary and more user-friendly front-end for dpkg is aptitude. |
| `apt` |	Apt provides a high-level command-line interface for the package management system. |
| `aptitude` |	Aptitude is an alternative to apt and is a high-level interface to the package manager. |
| `snap` |	Install, configure, refresh, and remove snap packages. Snaps enable the secure distribution of the latest apps and utilities for the cloud, servers, desktops, and the internet of things. |
| `gem` |	Gem is the front-end to RubyGems, the standard package manager for Ruby. |
| `pip` |	Pip is a Python package installer recommended for installing Python packages that are not available in the Debian archive. It can work with version control repositories (currently only Git, Mercurial, and Bazaar repositories), logs output extensively, and prevents partial installs by downloading all requirements before starting installation. |
| `git` |	Git is a fast, scalable, distributed revision control system with an unusually rich command set that provides both high-level operations and full access to internals. |

## Service and Process Management

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `systemctl` | Query or send control commands to the system manager. |
|  `kill <options> <PID>` | Use `kill -l` do show all available signals. |
| `bg` | Have a process running in the background |
| `jobs` | List all processes running in the background |
| `fg <id>` | Bring background process to the foreground and interact with it again. |
| `<command> &` | Run the command in background |
| `<command> ; <command>` | Run both commands regardless if the previous command was completed with or without errors. |
| `<command> && <command>` | Run the commands in order and if there is an error do not continue to the next command |

## File System Management

| Command   | Description                                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------|
| `fdisk`   | Main tool for disk management on Linux |
| `mount`   | Used to mount file systems on Linux |
| `umount`  | Unmount the file system |
| `lsof`    | List the open files on the file system |
| `mkswap`  | Set up a LInux swap area on a device or in a file |
| `swapon`  | Activate a swap area |
